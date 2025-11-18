# Craft Guidelines
- If the project uses Craft CMS, follow best practices for Craft development.
- Check which version of Craft is currently used.
    - If the version is <4 unique fields will have to be created.
    - If the version is >5 or higher fields can be reused. Check if a field with the desired settings already exists. If uncertain ask the user which field to (re)use.
- If creating a field for a large body of text ask if it should be a `CKeditor` (Craft <4) `Redactor` (Craft >5) or `Multi Line Plain Text` field.
- When create a component ask if it is a PageBuilder block or not.
    - If it is a PageBuilder block, place it in the `templates/_page-builder` folder.
    - The handle of the created EntryType should have the suffix `Block`. For example `contentMediaBlock`.
    - Add the created EntryType to the Matrix Field with handle `pageBuilder`/`commonPageBuilder`.
    - Don't use hardcoded content. Use Craft CMS variables and fields to populate content dynamically. You may refer to existing templates for examples of how to do this. If this fails you may use dummy variables and fields for illustration purposes.
- Most projects use the built-in Link Field Type, but some older projects use the plugin 
[Hyper](https://verbb.io/craft-plugins/hyper/docs/) or [Typed LinkField](https://github.com/sebastian-lenz/craft-linkfield). Check which field type is used and use the correct variables in the twig templates for getting values.
- You can find prompt instructions for Craft CMS development in the Plugin [Sidekick](https://github.com/doublesecretagency/craft-sidekick/tree/v1/src/prompts).