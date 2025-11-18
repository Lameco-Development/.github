# Tailwind CSS Guidelines
- If the project has a tailwind.config.js use Tailwind CSS v3 for styling
- If the project doesn't have a tailwind.config.js but instead has a theme.scss use Tailwind CSS v4 for styling:
    - Define design tokens (colors, spacing, radii, fonts, etc.) inside a global `@theme {}` block in your CSS. Example:
        ```css
        @theme {
            --color-primary: #FF0000;
            --color-secondary: #00FF00;
            --font-family-inter: 'Inter', sans-serif;
        }
        ```
- Use the generated utility classes (e.g., `bg-primary`, `text-secondary`, `font-inter`) instead of arbitrary values or custom classes.
- Only use arbitrary values (e.g., `bg-[var(--custom)]`) if a token is not defined in `@theme` and absolutely necessary.
- Think mobile first.
- For Typography use `typography.scss` and define font-sizes for desktop and mobile if provided in the variables from Figma.