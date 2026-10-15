# Angular HTTP & RxJS Knowledge

## HttpClient
### Setup
Use `provideHttpClient` in `app.config.ts`:
```typescript
export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(
      withFetch(), // Use Fetch API
      withInterceptors([...]) // Functional interceptors
    )
  ]
};
```

### Making Requests
Inject `HttpClient` and use methods like `get`, `post`, `put`, `delete`.
```typescript
http.get<User>('/api/user').subscribe(user => ...);
```
- **Type Safety**: Generic type `<T>` is an assertion, not validation.
- **Response Types**: 'json' (default), 'text', 'blob', 'arraybuffer'.

## RxJS Interop
### `toSignal`
Converts an Observable to a Signal.
```typescript
counter = toSignal(this.counter$, { initialValue: 0 });
```
- **Subscription**: Subscribes immediately, unsubscribes on destroy.
- **Options**: `initialValue`, `requireSync`, `manualCleanup`.

### `toObservable`
Converts a Signal to an Observable.
```typescript
query$ = toObservable(this.query);
```
- **Timing**: Asynchronous (uses effect).

### `rxResource` (Experimental)
Reactive resource based on RxJS stream.
```typescript
userResource = rxResource({
  params: () => ({ id: this.userId() }),
  stream: ({params}) => this.http.get(`/users/${params.id}`)
});
```

## Dependency Injection Patterns
- **Service Isolation**: Provide service in `@Component` providers to scope it to that component tree.
- **Specialized Providers**: Override services in child components for specialized behavior.
