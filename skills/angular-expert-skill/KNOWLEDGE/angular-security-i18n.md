# Angular Security & Internationalization Knowledge

## HttpClient Security
Built-in protection against common web vulnerabilities.

### XSSI Protection
Automatically strips `)]}',\n` prefix from JSON responses to prevent script inclusion attacks.

### XSRF/CSRF Protection
- **Mechanism**: Reads `XSRF-TOKEN` cookie and sets `X-XSRF-TOKEN` header.
- **Scope**: Applied to mutating requests (POST, PUT, DELETE, etc.) to relative URLs.
- **Configuration**:
```typescript
provideHttpClient(
  withXsrfConfiguration({
    cookieName: 'CUSTOM_XSRF_TOKEN',
    headerName: 'X-Custom-Xsrf-Header',
  })
)
```
- **Disable**: `withNoXsrfProtection()`

## Internationalization (i18n)
Process of adapting the application for different locales.
- **Internationalization**: Designing for global use.
- **Localization**: Building versions for specific locales (translation, data formatting).
- **Locale**: Determines formatting for dates, numbers, currencies, etc.
