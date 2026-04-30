# CodeSandbox Re-import Guide

## Recommended path

Create a new CodeSandbox from this project instead of reusing the broken `2222` workspace.

## Steps

1. Create a new sandbox in CodeSandbox.
2. Choose the `Universal` template.
3. Import this project from GitHub or upload the clean ZIP package.
4. Let CodeSandbox run `npm install`.
5. Run `npm run dev` if it does not start automatically.
6. Open port `5173` from the `Ports` panel.
7. Use the direct preview link in this form:

```text
https://<sandbox-id>-5173.csb.app/
```

## Verify JSON first

Open these links before opening the app root:

```text
https://<sandbox-id>-5173.csb.app/ctis_admin_published.json
https://<sandbox-id>-5173.csb.app/ctis_visible_site_dataset.json
```

If both links show JSON instead of HTML, then open:

```text
https://<sandbox-id>-5173.csb.app/
```

## Important notes

- Do not use port `2222`.
- Do not reuse the old broken workspace preview link.
- This project is configured so CodeSandbox should recognize `5173` as the primary preview port through `.codesandbox/tasks.json`.
