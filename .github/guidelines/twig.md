# Use Twig for HTML templates.
- When create a component ask if it is a PageBuilder block or not.
    - If it is a PageBuilder block, place it in the `templates/_page-builder` folder.
- Don't use hardcoded content. Use Craft CMS variables and fields to populate content dynamically. You may refer to existing templates for examples of how to do this. You may use dummy variables and fields for illustration purposes.
- Don't use tailwind classes with twig variables. Example to avoid: `<div class="bg-{{ item.color }}">`. Instead, use a twig variable with an array to map the classes. If it is a reusable twig variable map place it in `templates/_variables.twig`, otherwise place it in the same template file.
- Use macros instead of included partials if a block repeats or is used in a loop and is only required within the template. Example:
    ```twig
    {% macro card(item) %}
        <div class="card">{{ item.title }}</div>
    {% endmacro %}
    {% import _self as macros %}
    {% for item in items %}
        {{ macros.card(item) }}
    {% endfor %}
    ```
- Use included partials only for shared or cross-template components.
- Ask if a macro or partial is preferred when in doubt.
- Ask if a partial should be placed in the shared `partials` forder or inside the folder with the template the partial is only used in.