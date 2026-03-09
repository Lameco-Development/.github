# Craft 4 → Craft 5 Upgrade Guide

This is a reusable guide for upgrading any Craft 4 project to Craft 5. Each step ends with a commit.

## Step 0: Prerequisites

Before starting the upgrade, make sure the following are done:

1. Suspend all users except the admin user in the Craft control panel.

## Step 1: Create upgrade branch

Make sure you are on the latest `development` branch, import the production database, then create the upgrade branch:

```warp-runnable-command
git checkout development
```

```warp-runnable-command
git pull
```

```warp-runnable-command
dep lameco:db_download production
```

```warp-runnable-command
git checkout -b chore/update-craft5
```

Commit: n/a (no changes yet, branch is just created)

## Step 2: Verify HTTPS configuration

Craft 5 requires the dev server to run over HTTPS. Verify this is already set up.

1. Open `vite.config.js`
2. Check if `const httpsConfig` exists
   - **If it exists:** HTTPS is configured — no changes needed. Move to **Step 3**.
   - **If it does NOT exist:** continue with steps 2.3–2.5 below.
3. Add HTTPS support using Valet certificates. Add the following to the top of `vite.config.js`:

```js
import * as fs from "fs";
import * as os from "os";
import * as path from "path";
const host = path.basename(process.cwd()) + ".test";
const valetCertDir = `${os.homedir()}/.config/valet/Certificates`;
const certPath = `${valetCertDir}/${host}.crt`;
const keyPath = `${valetCertDir}/${host}.key`;
const httpsConfig =
  fs.existsSync(certPath) && fs.existsSync(keyPath)
    ? { key: fs.readFileSync(keyPath), cert: fs.readFileSync(certPath) }
    : true;
```

4. Update the `server` block in `vite.config.js`:

```js
server: {
    host: "0.0.0.0",
    https: httpsConfig,
    origin: "http://localhost:3000",
    port: 3000,
    strictPort: true,
}
```

5. In `config/vite.php`, update `devServerPublic` to:

```php
'devServerPublic' => App::env('PRIMARY_SITE_URL') ? App::env('PRIMARY_SITE_URL') . ':3000/' : 'https://localhost:3000/',
```

6. Update `.env.example` and `.env`: change `PRIMARY_SITE_URL` from `http://` to `https://`.
7. Run `valet secure` to generate the SSL certificates.
8. Commit:

```warp-runnable-command
git add vite.config.js config/vite.php .env.example .env
git commit -m "chore: configure HTTPS for Craft 5 dev server"
```

If no changes were needed, skip the commit for this step.

## Step 3: Set Node version and reinstall dependencies

1. Open `.nvmrc` and verify it contains `v24.12.0`.
   - **If it does:** no changes needed — skip the rest of this step.
   - **If it does NOT:** update `.nvmrc` to `v24.12.0`, then continue with steps 3.2–3.4 below.
2. Run the following commands:

```warp-runnable-command
rm -rf node_modules
```

```warp-runnable-command
nvm use
```

```warp-runnable-command
yarn
```

3. Commit:

```warp-runnable-command
git add .nvmrc
git commit -m "chore: set Node version to v24.12.0"
```

## Step 4: Update to latest Craft 4 and packages

Before upgrading to Craft 5, make sure all packages are on their latest minor versions.

1. Run `composer update` to update all packages to their latest minor versions.
2. Run `php craft update all` to apply any remaining Craft CMS and plugin updates.
3. Commit:

```warp-runnable-command
git add composer.json composer.lock
git commit -m "chore: update all packages to latest minor versions"
```

## Step 5: Rebuild project config

1. Run `php craft project-config/rebuild`.
2. Commit:

```warp-runnable-command
git add config/project
git commit -m "chore: rebuild project config"
```

## Step 6: Fix field layout UIDs

1. Run `php craft utils/fix-field-layout-uids`.
2. Commit:

```warp-runnable-command
git add config/project
git commit -m "chore: fix field layout UIDs"
```

## Step 7: Apply project config

1. Run `php craft project-config/apply`.
2. Commit:

```warp-runnable-command
git add config/project
git commit -m "chore: apply project config"
```

## Step 8: Update composer.json for Craft 5

Some plugins are abandoned or not available for Craft 5 and need to be removed/migrated.

1. Check if `born05/craft-twofactorauthentication` exists in `composer.json`. If so, remove it — Craft 5 handles 2FA natively.
   - **⚠️ Reminder:** After the upgrade and migrations are complete, configure 2FA in Craft 5's built-in settings.
2. Check if `vaersaagod/matrixmate` exists in `composer.json`. If so, remove it.
   - **⚠️ Reminder:** After the upgrade and migrations are complete, create a migration to handle MatrixMate functionality.
3. Bump `craftcms/generator` from `^1.3.0` to `^2.0` in `composer.json`.
4. Update `craftcms/cms` to `"^5.0.0"` in `composer.json`.
5. Change all exact-pinned plugin versions to `*` so Composer can resolve them to the latest Craft 5-compatible versions. For example:
   - `"craftcms/feed-me": "5.14.0"` → `"craftcms/feed-me": "*"`
   - `"verbb/formie": "^2.2.14"` → `"verbb/formie": "*"`
   - Do this for **all** plugins (not for non-plugin packages like `vlucas/phpdotenv` or `erusev/parsedown`).
   - **Exception — `sebastianlenz/linkfield`:** The stable releases of this plugin only support Craft 4. Set it explicitly to `"3.0.0-beta"` instead of `"*"`, as that is the Craft 5-compatible beta release:
     `"sebastianlenz/linkfield": "3.0.0-beta"`
6. Check if `lameco/craft-twig-components` exists in `composer.json` with a version lower than `1.x`. If so, add the following inline aliases to `composer.json` under `require` to satisfy its dependencies:
   ```json
   "performing/twig-components": "0.7.0 as 0.6.9",
   "voku/portable-utf8": "dev-master#c4b3774 as 6.0.13",
   ```
7. Run `composer update -W` (the `-W` flag allows upgrades/downgrades of dependencies to resolve version conflicts).
8. After the update succeeds, pin all plugin versions back to their resolved versions from `composer.lock` to prevent unintended future updates. Run the following script to do this automatically:

```bash
php -r "
\$lock = json_decode(file_get_contents('composer.lock'), true);
\$json = json_decode(file_get_contents('composer.json'), true);
\$versions = [];
foreach (array_merge(\$lock['packages'], \$lock['packages-dev']) as \$pkg) {
    \$versions[\$pkg['name']] = ltrim(\$pkg['version'], 'v');
}
foreach (['require', 'require-dev'] as \$section) {
    foreach (\$json[\$section] ?? [] as \$pkg => \$constraint) {
        if (\$constraint === '*' && isset(\$versions[\$pkg])) {
            \$json[\$section][\$pkg] = \$versions[\$pkg];
        }
    }
}
file_put_contents('composer.json', json_encode(\$json, JSON_PRETTY_PRINT | JSON_UNESCAPED_SLASHES) . PHP_EOL);
echo 'Done.' . PHP_EOL;
"
```

- This only pins packages that were set to `*` — range constraints like `^5.0.0` are left untouched.

9. Run `php craft up` to apply all migrations.
10. Commit only the relevant files — **do not use `git add -A`** as it would accidentally include the upgrade guide and other untracked files:

```warp-runnable-command
git add config/ modules/ composer.json composer.lock
git commit -m "chore: upgrade to Craft 5"
```

## Step 9: Fix custom module compatibility

Craft 5 introduced breaking changes in several base classes. Audit **all PHP files in `modules/`** — including fields, controllers, services, and any other custom classes — for compatibility issues.

### Audit all PHP files in `modules/`

Start with a broad search across the entire `modules/` directory to spot common Craft 5 breaking patterns:

```bash
# Find deprecated property declarations
grep -rn "public bool \$optgroups" modules/

# Find any class that extends a Craft base class (good starting point for manual review)
grep -rn "extends craft\\" modules/
```

Manually review the results and check each file against the Craft 5 upgrade notes:
👉 https://craftcms.com/docs/5.x/extend/updating-plugins.html

### `craft\fields\Dropdown` — `$optgroups` is now `static`

If any custom field in `modules/` extends `craft\fields\Dropdown` and overrides the `$optgroups` property, it must be declared as `static` to match the parent class:

```php
// Before (Craft 4)
public bool $optgroups = false;

// After (Craft 5)
public static bool $optgroups = false;
```

Fix **every** file with `public bool $optgroups` by adding `static`.

### Controllers, services, and other classes

Also check:

- **Controllers** (`modules/*/controllers/`) — verify any overridden methods still match the Craft 5 base controller signatures.
- **Services** — check for deprecated service methods or changed return types.
- **Events** — event class names and properties may have changed; search for `Event::on(` and verify event class imports are still valid.

```bash
# Check for potentially moved or renamed Craft classes
grep -rn "use craft\\" modules/ | grep -v "vendor/"
```

Commit after fixing all issues:

```warp-runnable-command
git add modules/
git commit -m "fix: update custom module compatibility for Craft 5"
```

## Step 10: Update environment-label config

If `topshelfcraft/environment-label` is installed, the `targetSelector` in `config/environment-label.php` must be updated — the Craft 4 sidebar selector no longer exists in Craft 5.

1. Open `config/environment-label.php`.
2. Change `targetSelector` from `'#global-sidebar:before'` to `'body:before'`:

```php
'targetSelector' => 'body:before',
```

3. Commit:

```warp-runnable-command
git add config/environment-label.php
git commit -m "fix: update environment-label targetSelector for Craft 5"
```

## Step 11: Migrate MatrixMate to native Craft 5 entry type filtering

If `vaersaagod/matrixmate` was installed and `config/matrixmate.php` exists, it must be migrated to a custom module event handler — MatrixMate is not compatible with Craft 5.

**If `config/matrixmate.php` does not exist, skip this step.**

In Craft 5.8+, entry type **grouping** (labels like "Content Blocks") is natively supported via the Matrix field layout in the CP. What still requires code is **site-specific filtering** (showing different entry types per site), which was handled by `hideUngroupedTypes` in MatrixMate.

1. Open `config/matrixmate.php` and note all field handles, sites, and allowed entry type handles.
2. In `modules/lameco/Module.php`, add the following imports:

```php
use craft\events\DefineEntryTypesForFieldEvent;
use craft\fields\Matrix;
use craft\models\EntryType;
```

3. Add a `Matrix::EVENT_DEFINE_ENTRY_TYPES` event handler inside `attachEventHandlers()`. Use the site handle from `$event->element?->getSite()?->handle` to filter entry types per site:

```php
Event::on(Matrix::class, Matrix::EVENT_DEFINE_ENTRY_TYPES, static function (DefineEntryTypesForFieldEvent $event) {
    $field = $event->sender;
    if ($field?->handle !== 'yourFieldHandle') {
        return;
    }

    $siteHandle = $event->element?->getSite()?->handle ?? '';

    $allowedTypes = match ($siteHandle) {
        'yourSiteHandle' => ['entryTypeA', 'entryTypeB'],
        default          => ['entryTypeA', 'entryTypeC'],
    };

    $event->entryTypes = array_values(array_filter(
        $event->entryTypes,
        static fn(EntryType $entryType) => in_array($entryType->handle, $allowedTypes, true)
    ));
});
```

4. **(Optional) Add native entry type grouping in Craft 5.8+**
   If the MatrixMate config had groups, use native Craft 5 grouping instead. Groups are stored as a per-field usage config — they go in the **field's** YAML (`config/project/fields/<fieldHandle>--<uid>.yaml`), not in the entry type YAML.

   Find the field's YAML and add a `group` key to **every** entry type inside `settings.entryTypes`. The `group` value must be set on each entry type individually — entry types without an explicit `group` will appear under a "General" catch-all group.

   The project config YAML uses `__assoc__` arrays. Add `group` as a second key-value pair alongside `uid` for every entry type:

   ```yaml
   settings:
     entryTypes:
       -
         __assoc__:
           -
             - uid
             - <entryTypeUid> # firstBlockInGroupA
           -
             - group
             - 'Group A Label'
       -
         __assoc__:
           -
             - uid
             - <entryTypeUid> # secondBlockInGroupA
           -
             - group
             - 'Group A Label'
       -
         __assoc__:
           -
             - uid
             - <entryTypeUid> # firstBlockInGroupB
           -
             - group
             - 'Group B Label'
   ```

   > **⚠️ Important:** Every entry type must have `group` set. If you only set it on the first entry in a group, the rest will fall into a separate "General" group in the CP.

   Then run `php craft project-config/apply` to apply the changes.

5. Delete `config/matrixmate.php` as it is no longer used.
6. Commit:

```warp-runnable-command
git add modules/lameco/Module.php config/project/
git rm config/matrixmate.php
git commit -m "feat: migrate MatrixMate entry type filtering to custom module event handler"
```

## Step 12: Migrate Redactor to CKEditor

If `craftcms/redactor` is NOT in `composer.json`, skip this step.

1. Add `craftcms/ckeditor` to `composer.json` and install it:

```warp-runnable-command
php -r "
\$data = json_decode(file_get_contents('composer.json'), true);
\$req = [];
foreach (\$data['require'] as \$k => \$v) {
    if (\$k === 'craftcms/feed-me') \$req['craftcms/ckeditor'] = '*';
    \$req[\$k] = \$v;
}
\$data['require'] = \$req;
file_put_contents('composer.json', json_encode(\$data, JSON_PRETTY_PRINT | JSON_UNESCAPED_SLASHES) . PHP_EOL);
echo 'Done' . PHP_EOL;
"
```

```warp-runnable-command
composer update craftcms/ckeditor --no-interaction
```

```warp-runnable-command
php craft plugin/install ckeditor
```

Then pin the version back using the auto-pin script from Step 8.

2. Run the official CKEditor conversion command. For each unique Redactor config, it will automatically create a new CKEditor config and associate it with the appropriate fields:

```warp-runnable-command
php craft ckeditor/convert/redactor
```

3. Run `php craft project-config/apply` to apply the converted fields and configs.

3b. Update the auto-generated CKEditor configs to match the starter kit. Since CKEditor v5.x,
the configuration is no longer stored in separate config files but is embedded directly in each
field's YAML under `config/project/fields/`.

> **⚠️ Important:** The converter migrates settings based on the **actual Redactor config names** in the
> project. These may not match the starter kit names "Title", "Default", and "Simple". For example,
> a project may have had a "Typography" Redactor config instead of "Default".
>
> **What to do:**
> 1. Find the CKEditor fields in `config/project/fields/` and inspect each file's `settings.toolbar:` value.
> 2. For each field, determine which starter kit config applies based on its purpose:
>    - A field used on **title/heading content** with only `bold` → apply **Title** settings
>    - A field used on **simple content** with `heading`, `bold`, `italic`, `link` → apply **Simple** settings
>    - A field used on **rich content** with the full toolbar → apply **Default** settings
> 3. Replace the relevant keys inside `settings:` with the corresponding starter kit values below.

Open each CKEditor field file under `config/project/fields/` and update the `settings:` block
to match one of the following starter kit configs:

**Title** — fields used for title/heading content:

```yaml
settings:
    headingLevels:
        - 2
        - 3
        - 4
        - 5
        - 6
    toolbar:
        - bold
```

**Simple** — fields used for lightweight rich text:

```yaml
settings:
    css: ".text-intro {\r\n  font-size: 120%;\r\n  line-height: 140%;\r\n}"
    headingLevels:
        - 2
        - 3
        - 4
        - 5
        - 6
    options:
        link:
            addTargetToExternalLinks: false
            decorators:
                openInNewTab:
                    attributes:
                        rel: 'noopener noreferrer'
                        target: _blank
                    label: 'Open in a new tab'
                    mode: manual
        style:
            definitions:
                -
                    classes:
                        - text-intro
                    element: p
                    name: Intro
    toolbar:
        - heading
        - style
        - '|'
        - bold
        - italic
        - link
```

**Default** — fields used for full rich text editing:

```yaml
settings:
    css: ".text-intro {\r\n  font-size: 120%;\r\n  line-height: 140%;\r\n}"
    headingLevels:
        - 2
        - 3
        - 4
        - 5
        - 6
    options:
        link:
            addTargetToExternalLinks: false
            decorators:
                openInNewTab:
                    attributes:
                        rel: 'noopener noreferrer'
                        target: _blank
                    label: 'Open in a new tab'
                    mode: manual
        style:
            definitions:
                -
                    classes:
                        - text-intro
                    element: p
                    name: Intro
    toolbar:
        - heading
        - style
        - '|'
        - bold
        - underline
        - italic
        - strikethrough
        - link
        - subscript
        - superscript
        - '|'
        - bulletedList
        - numberedList
        - outdent
        - indent
```

4. Remove the Redactor plugin:

```warp-runnable-command
php craft plugin/uninstall redactor
````

Remove `craftcms/redactor` from `composer.json`, then run:

```warp-runnable-command
composer update craftcms/redactor --no-interaction
```

5. Commit:

```warp-runnable-command
git add composer.json composer.lock config/project/
git commit -m "feat: migrate Redactor to CKEditor"
```

## Step 13: Run Super Table migration

If `verbb/super-table` is NOT in `composer.json`, skip this step.

Super Table is not supported in Craft 5. Run the migration command to convert Super Table fields to native Matrix fields, then remove the plugin.

1. Run the Super Table migration command:

```warp-runnable-command
php craft super-table/migrate
```

2. Uninstall the plugin and remove it from `composer.json`:

```warp-runnable-command
php craft plugin/uninstall super-table
```

Remove `verbb/super-table` from `composer.json`, then run:

```warp-runnable-command
composer update verbb/super-table --no-interaction
```

3. Commit:

```warp-runnable-command
git add composer.json composer.lock config/project/
git commit -m "chore: migrate Super Table fields to native Matrix and remove plugin"
```

## Step 14: Require two-step verification for all users

In Craft 5, two-factor authentication is built in. Enable it for all users by updating `config/project/users/users.yaml` — add `require2fa: all`:

```yaml
allowPublicRegistration: false
defaultGroup: null
photoSubpath: null
photoVolumeUid: null
require2fa: all
requireEmailVerification: true
```

Then apply and commit:

```warp-runnable-command
php craft project-config/apply
```

```warp-runnable-command
git add config/project/users/users.yaml
git commit -m "chore: require two-step verification for all users"
```

## Step 15: Update Formie templates for v3

If `verbb/formie` is NOT in `composer.json`, skip this step.

Formie 3 (included with Craft 5) has breaking changes for templates and theme config.

### 1. Rename `fieldInputContainer` → `fieldInputWrapper` in theme config

The `fieldInputContainer` key in the `themeConfig` has been renamed to `fieldInputWrapper`. Also, the CSS class `.fui-input-container` has been renamed to `.fui-input-wrapper`.

Search for `fieldInputContainer` in your templates (typically `templates/variables.twig`) and rename every occurrence to `fieldInputWrapper`. Also search for `.fui-input-container` in your CSS/SCSS and rename to `.fui-input-wrapper`.

### 2. Update field iteration in templates (if applicable)

If any template iterates over form or submission fields, update the method calls:

| Formie v2                                       | Formie v3                |
| ----------------------------------------------- | ------------------------ |
| `form.getCustomFields()`                        | `form.getFields()`       |
| `submission.getFieldLayout().getCustomFields()` | `submission.getFields()` |
| `page.getCustomFields()`                        | `page.getFields()`       |
| `row.fields`                                    | `row.getFields()`        |

### 3. Commit

```warp-runnable-command
git add templates/
git commit -m "fix: update Formie templates for v3"
```

## Step 16: Migrate sebastianlenz/linkfield to native Craft Link field

If `sebastianlenz/linkfield` is NOT in `composer.json`, skip this step.

The Typed Link Field plugin is abandoned and not compatible with Craft 5. Copy the migration script below into `migrations/m250305_135052_typed_link_field_to_craft_link_field.php`. It dynamically reads each field's original Lenz settings (enabled types, label field, target) and maps them faithfully to the native Craft Link field — no hardcoded field names, so it works for any project.

> Based on: https://github.com/sebastian-lenz/craft-linkfield/issues/286#issuecomment-2720037030 (corrected and improved)

```php
<?php

namespace craft\contentmigrations;

use Craft;
use craft\base\ElementInterface;
use craft\base\FieldInterface;
use craft\db\Migration;
use craft\db\Query;
use craft\fields\Link;
use craft\helpers\Db;
use craft\helpers\Json;
use craft\helpers\App;

/**
 * m250305_135052_typed_link_field_to_craft_link_field migration.
 *
 * See: https://github.com/sebastian-lenz/craft-linkfield/issues/286#issuecomment-2720037030
 */
class m250305_135052_typed_link_field_to_craft_link_field extends Migration
{
    public array $allowedTypes = [
        'asset',
        'category',
        'email',
        'entry',
        'url',
        'tel',
        'custom',
    ];

    /**
     * @inheritdoc
     * @throws \JsonException
     */
    public function safeUp(): bool
    {
        App::maxPowerCaptain();

        // Get all Lenz LinkField fields
        $fields = (new Query())
            ->from('{{%fields}}')
            ->where(['type' => 'typedlinkfield\\fields\\LinkField'])
            ->orWhere(['type' => 'lenz\\linkfield\\fields\\LinkField'])
            ->all();

        // Loop through each field and migrate settings in this pass
        foreach ($fields as $field) {
            echo "Preparing to migrate field \"{$field['handle']}\" ({$field['uid']}) settings.\n";

            // Read the original Lenz typeSettings to preserve per-field configuration
            $lenzSettings = Json::decode($field['settings']) ?? [];
            $lenzTypeSettings = $lenzSettings['typeSettings'] ?? [];

            // Map Lenz type names → native Craft Link types.
            // 'custom' is a free-form URL in Lenz and maps to 'url' in Craft.
            $typeMapping = [
                'entry'    => 'entry',
                'asset'    => 'asset',
                'category' => 'category',
                'email'    => 'email',
                'tel'      => 'tel',
                'url'      => 'url',
                'custom'   => 'url',
            ];

            // Build the types list, preserving only what was enabled originally
            $enabledTypes = [];
            foreach ($typeMapping as $lenzType => $craftType) {
                if (!empty($lenzTypeSettings[$lenzType]['enabled'])) {
                    $enabledTypes[$craftType] = true;
                }
            }
            $types = array_values(array_keys($enabledTypes));

            // Fall back to all types if nothing could be determined
            if (empty($types)) {
                $types = ['entry', 'url', 'asset', 'category', 'email', 'tel'];
            }

            // Map allowCustomText → showLabelField, allowTarget → advancedFields
            $showLabelField = !empty($lenzSettings['allowCustomText']);
            $allowTarget = !empty($lenzSettings['allowTarget']);

            $nativeFieldSettings = [
                'advancedFields' => $allowTarget ? ['target'] : [],
                'fullGraphqlData' => true,
                'maxLength' => 255,
                'showLabelField' => $showLabelField,
                'typeSettings' => [
                    'entry' => [
                        'sources' => '*'
                    ],
                    'url' => [
                        'allowRootRelativeUrls' => '1',
                        'allowAnchors' => '1'
                    ],
                    'asset' => [
                        'sources' => '*',
                        'allowedKinds' => '*',
                        'showUnpermittedVolumes' => '',
                        'showUnpermittedFiles' => ''
                    ],
                    'category' => [
                        'sources' => '*'
                    ]
                ],
                'types' => $types,
            ];

            echo "  Enabled types: " . implode(', ', $types) . "\n";

            // Update the type and settings for typedlinkfield fields to native Craft Link field equivalents
            $this->update(
                '{{%fields}}',
                [
                    'type' => Link::class,
                    'settings' => json_encode($nativeFieldSettings, JSON_THROW_ON_ERROR)
                ],
                ['uid' => $field['uid']]
            );

            echo "> Field \"{$field['handle']}\" settings migrated.\n\n";
        }

        // now iterate through the content and migrate it using the new field settings
        $fields = (new Query())
            ->from('{{%fields}}')
            ->where(['type' => Link::class])
            ->all();

        foreach ($fields as $field) {
            // Migrate content
            echo "Preparing to migrate field \"{$field['handle']}\" ({$field['uid']}) content.\n";
            $fieldModel = Craft::$app->getFields()->getFieldById($field['id']);

            if ($fieldModel) {
                // Get content from the lenz_linkfield table
                $contentRows = (new Query())
                    ->select(['*'])
                    ->from('{{%lenz_linkfield}}')
                    ->where(['fieldId' => $fieldModel['id']])
                    ->all();

                if (count($contentRows) < 1) {
                    echo "> No content to migrate for field '{$field['handle']}'\n";
                    continue;
                }

                if ($fieldModel->context === 'global') {
                    foreach ($contentRows as $row) {
                        $settings = $this->convertLinkContent($fieldModel, $row);

                        $element = Craft::$app->getElements()->getElementById($row['elementId'], null, $row['siteId']);
                        if ($element) {
                            if ($settings) {
                                $newContent = $this->getElementContentForField($element, $fieldModel, $settings);

                                Db::update('{{%elements_sites}}', ['content' => $newContent], ['elementId' => $row['elementId'], 'siteId' => $row['siteId']]);

                                echo "    > Migrated content for element #{$row['elementId']}\n";
                            } else {
                                if ($settings !== null) {
                                    echo "    > Unable to convert content for element #{$row['elementId']}\n";
                                }
                            }
                        } else {
                            echo "    > Unable to find element #{$row['elementId']} and site #{$row['siteId']}\n";
                        }
                    }
                }
            }
            echo "> Field \"{$field['handle']}\" content migrated.\n\n";
        }

        return true;
    }

    /**
     * @inheritdoc
     */
    public function safeDown(): bool
    {
        echo "m250305_135052_typed_link_field_to_craft_link_field cannot be reverted.\n";
        return false;
    }

    public function convertLinkContent($field, array $settings): bool|array|null
    {
        $linkType = $settings['type'] ?? null;

        if (!$linkType) {
            return null;
        }

        if (!in_array($linkType, $this->allowedTypes)) {
            return false;
        }

        $advanced = Json::decode($settings['payload']);
        $linkValue = $settings['linkedUrl'] ?? null;
        $linkText = $advanced['customText'] ?? null;
        $linkSiteId = $settings['siteId'] ?? null;
        $linkId = $settings['linkedId'] ?? null;
        $linkTarget = $advanced['target'] ?? null;

        if (($linkType === 'entry' || $linkType === 'asset' || $linkType === 'category') && (!$linkId || !$linkSiteId)) {
            return false;
        }

        if ($linkType === 'entry') {
            $linkValue = "{entry:{$linkId}@{$linkSiteId}:url}";
        } elseif ($linkType === 'asset') {
            $linkValue = "{asset:{$linkId}@{$linkSiteId}:url}";
        } elseif ($linkType === 'category') {
            $linkValue = "{category:{$linkId}@{$linkSiteId}:url}";
        } elseif ($linkType === 'email') {
            $linkValue = 'mailto:' . $linkValue;
        } elseif ($linkType === 'tel') {
            $linkValue = 'tel:' . $linkValue;
        } elseif ($linkType === 'custom') {
            $linkType = 'url';
        }

        return [
            'value' => $linkValue,
            'type' => $linkType,
            'label' => $linkText,
            'target' => $linkTarget,
        ];
    }

    // From:
    // https://github.com/verbb/hyper/blob/craft-5/src/migrations/PluginContentMigration.php#L141
    protected function getElementContentForField(ElementInterface $element, FieldInterface $field, array $fieldValue): array
    {
        $fieldContent = [];

        // Get the field content as JSON, indexed by field layout element UID
        if ($fieldLayout = $element->getFieldLayout()) {
            foreach ($fieldLayout->getCustomFields() as $fieldLayoutField) {
                $sourceHandle = $fieldLayoutField->layoutElement?->getOriginalHandle() ?? $fieldLayoutField->handle;

                if ($field->handle === $sourceHandle) {
                    $fieldContent[$fieldLayoutField->layoutElement->uid] = $fieldValue;
                }
            }
        }

        // Fetch the current JSON content so we can merge in the new field content
        $oldContent = Json::decode((new Query())
            ->select(['content'])
            ->from('{{%elements_sites}}')
            ->where(['elementId' => $element->id, 'siteId' => $element->siteId])
            ->scalar() ?? '') ?? [];

        // Another sanity check just in cases where content is double encoded
        if (is_string($oldContent) && Json::isJsonObject($oldContent)) {
            $oldContent = Json::decode($oldContent);
        }

        return array_merge($oldContent, $fieldContent);
    }
}
```

1. Run an AI agent to inspect the actual Lenz field configurations in this project and update the `typeMapping` and `allowedTypes` in the migration to match:

```warp-runnable-command
claude "Read migrations/m250305_135052_typed_link_field_to_craft_link_field.php. Then run the following PHP snippet to inspect the actual Lenz fields and all distinct type values used in content:

php -r \"
\\\$db = new PDO('mysql:host=127.0.0.1;dbname=' . getenv('CRAFT_DB_DATABASE'), getenv('CRAFT_DB_USER'), getenv('CRAFT_DB_PASSWORD'));
\\\$fields = \\\$db->query(\\\"SELECT handle, settings FROM craft_fields WHERE type IN ('typedlinkfield\\\\\\\fields\\\\\\\LinkField','lenz\\\\\\\linkfield\\\\\\\fields\\\\\\\LinkField')\\\")->fetchAll(PDO::FETCH_ASSOC);
foreach (\\\$fields as \\\$f) {
    \\\$s = json_decode(\\\$f['settings'], true);
    echo \\\$f['handle'] . ': enabled types = ' . implode(', ', array_keys(array_filter(\\\$s['typeSettings'] ?? [], fn(\\\$t) => !empty(\\\$t['enabled'])))) . PHP_EOL;
}
\\\$types = \\\$db->query('SELECT DISTINCT type FROM craft_lenz_linkfield')->fetchAll(PDO::FETCH_COLUMN);
echo 'Content types in lenz_linkfield table: ' . implode(', ', \\\$types) . PHP_EOL;
\"

Based on the output: update the \$typeMapping array so every type name found in the fields settings and the lenz_linkfield table has an explicit mapping to a native Craft Link type (entry, asset, category, url, email, tel). Also update \$allowedTypes to include every type that appears in the content. If any type names are custom/unknown, map them to the closest native type and add a comment explaining the mapping."
```

2. Run the migration:

```warp-runnable-command
php craft migrate/up m250305_135052_typed_link_field_to_craft_link_field
```

3. Rebuild project config to capture the field type changes:

```warp-runnable-command
php craft project-config/rebuild
```

4. Remove the `lenz_linkfield` table from the database:

```warp-runnable-command
mysql -u root -proot <database_name> -e "DROP TABLE IF EXISTS lenz_linkfield;"
```

> **Note:** Replace `root`/`root` and `<database_name>` with your actual DB credentials from `.env` (`CRAFT_DB_USER`, `CRAFT_DB_PASSWORD`, `CRAFT_DB_DATABASE`). Craft does not have a built-in `db/run-sql` command.

5. Remove the migration file:

```warp-runnable-command
rm migrations/m250305_135052_typed_link_field_to_craft_link_field.php
```

6. Uninstall the plugin and remove it from `composer.json`:

```warp-runnable-command
php craft plugin/uninstall typedlinkfield
```

Remove `sebastianlenz/linkfield` from `composer.json`, then run:

```warp-runnable-command
composer update sebastianlenz/linkfield --no-interaction
```

7. Update templates — search for all Link field usages and update the API calls:

| Old (Typed Link Field)        | New (native Craft Link) |
| ----------------------------- | ----------------------- |
| `%field%.isEmpty()`           | `%field% is empty`      |
| `not %field%.isEmpty()`       | `%field% is not empty`  |
| `not %field%.isEmpty` (no ()) | `%field% is not empty`  |
| `%field%.url()`               | `%field%.url`           |
| `%field%.text()`              | `%field%.label`         |

> **Note:** Twig property access (`.isEmpty` without parens) will also call `isEmpty()` under the hood. Remove any remaining `and not %field%.isEmpty` conditions — they are redundant if `%field% is not empty` is already checked in the same condition.

8. Commit:

```warp-runnable-command
git add migrations/ templates/ composer.json composer.lock config/project/
git commit -m "feat: migrate Typed Link fields to native Craft Link field"
```

---

## Step 17: Fix templates for Craft 5 compatibility

Craft 5 changes how several APIs work in Twig templates. After upgrading, audit all templates for the following patterns.

### 1. Element URLs: method → property

In Craft 4, calling `.url()` as a method on elements (entries, categories, assets) worked via Twig's magic method resolution. In Craft 5 this is no longer supported.

**Find & replace:**

```
.url()  →  .url
.getUrl()  →  .url
```

Run in the project root:

```bash
grep -rn "\.url()\|\.getUrl()" templates/ --include="*.twig"
```

### 2. Link field API changes

If you migrated from `sebastianlenz/linkfield` (Step 16 covers this), also check for these Link-field-specific patterns:

| Old                     | New                    |
| ----------------------- | ---------------------- |
| `%field%.url()`         | `%field%.url`          |
| `%field%.text()`        | `%field%.label`        |
| `%field%.isEmpty()`     | `%field% is empty`     |
| `not %field%.isEmpty()` | `%field% is not empty` |

### 3. Matrix fields (formerly Super Table)

If the project used the **Super Table** plugin in Craft 4, those fields were converted to native **Matrix** fields during the Craft 5 upgrade. In Craft 4 Super Table with `maxRows: 1`, templates could access sub-fields directly:

```twig
{# Craft 4 Super Table — worked directly #}
{% if entry.socialMedia.linkedin is not empty %}
```

In Craft 5, Matrix fields return an `EntryQuery`. You must call `.one()` first to get the nested entry:

```twig
{# Craft 5 Matrix — call .one() first #}
{% set sm = entry.socialMedia.one() %}
{% if sm and sm.linkedin is not empty %}
```

**Audit:** search for Matrix field handles followed directly by a sub-field name. Look for patterns where a field that holds sub-fields is accessed as if it's a single object:

```bash
# Identify potential Matrix direct-access patterns (replace 'socialMedia' with your field handles)
grep -rn "\.socialMedia\." templates/ --include="*.twig"
```

Also check `variables.twig` or any base template where "showX" flags may be computed — these often contain the same stale direct-access patterns.

### 4. Quick audit commands

Run these to check for any remaining Craft 4 patterns before deploying:

```bash
# Should return no results
grep -rn "\.url()\|\.getUrl()\|\.text()\|\.isEmpty()" templates/ --include="*.twig"
```

### 5. Apply fixes with a one-liner

If there are still hits, you can bulk-replace with Python:

```bash
python3 << 'EOF'
import os, re, glob

files = glob.glob('templates/**/*.twig', recursive=True)
for path in files:
    with open(path) as f:
        original = f.read()
    content = original
    content = re.sub(r'not ([\w.]+)\.isEmpty\(\)', r'\1 is not empty', content)
    content = re.sub(r'([\w.]+)\.isEmpty\(\)', r'\1 is empty', content)
    content = content.replace('.url()', '.url')
    content = content.replace('.text()', '.label')
    content = content.replace('.getUrl()', '.url')
    if content != original:
        with open(path, 'w') as f:
            f.write(content)
        print(f"Updated: {path}")
EOF
```

> **Note 1:** After running the script, manually check for redundant conditions like `%field% is not empty and not %field%.isEmpty` — remove the `and not %field%.isEmpty` part as it is now a plain property access that still calls the (removed) `isEmpty()` method via Twig.

> **Note 2:** Also check for `.text` and `.url` (no parens) used on link fields — Twig resolves `button.text` as a call to `getText()` or `text()`, which will fail. The script above only handles `()` variants. Run this extra check:
>
> ```bash
> grep -rn "\.text\b" templates/ --include="*.twig"
> ```
>
> Replace any `%linkfield%.text` → `%linkfield%.label` that were not caught by the script.

### 6. Commit

```warp-runnable-command
git add templates/
git commit -m "fix: update templates for Craft 5 compatibility"
```

---

## Step 18: Silence Bootstrap/Sass deprecation warnings in Vite

> **Only applies if:** the project uses Bootstrap SCSS and Sass 1.65 or newer.

Modern versions of Sass (1.65+) deprecated global color functions (`red()`, `green()`, `blue()`, etc.) that Bootstrap 5.x still uses internally. This results in hundreds of deprecation warnings in the terminal during `yarn dev` / `yarn build`, but does **not** break the build.

This is a **known Bootstrap issue** — the Bootstrap team is aware and working on a fix for a future release. In the meantime, they officially recommend silencing the warnings in your Vite config. See their own documentation:
👉 https://getbootstrap.com/docs/5.3/getting-started/vite/ (section "Configure Vite")
👉 Open issue: https://github.com/twbs/bootstrap/issues/40962

Add a `css` block to `vite.config.js`:

```js
css: {
    preprocessorOptions: {
        scss: {
            silenceDeprecations: ['color-functions', 'global-builtin', 'import', 'if-function'],
        },
    },
},
```

- `silenceDeprecations` — silences specific Sass deprecation categories; the four listed above cover all warnings Bootstrap 5 currently triggers.

After adding this, restart `yarn dev` — the warnings should be gone.

---

## Step 19: Re-run Feed Me feeds (if Feed Me is installed)

> **Only applies if:** `craftcms/feed-me` is in `composer.json`

After the upgrade, check if Feed Me feeds must be run manually at least once to re-index content against the new Craft 5 data structures.

1. Log in to the Craft control panel.
2. Go to **Feed Me → Feeds**.
3. Run each feed individually by clicking **Run feed**.
4. Verify the feed completed without errors and that content was imported correctly.

> **Note:** If feeds are scheduled via cron or queue jobs, verify those are still configured and pointing to the correct endpoint after the upgrade.

---

## Step 20: Verify Blitz cache query string params (if Blitz is installed)

> **Only applies if:** `putyourlightson/craft-blitz` is in `composer.json`

Blitz's query string parameter configuration may differ between environments. After the upgrade, compare the local Blitz settings against the production configuration to make sure no params are missing or incorrect.

1. Go to **Blitz → Settings → Query String Caching** on both local and production.
2. Compare the **Included query string params** and **Excluded query string params** lists.
3. Update the production configuration to match any changes introduced during the upgrade, or vice versa if production values should be the source of truth.
4. Clear and warm the Blitz cache after any changes:

```warp-runnable-command
php craft blitz/cache/clear && php craft blitz/cache/warm
```
