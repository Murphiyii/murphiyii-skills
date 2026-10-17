# Angular Animations & Harnesses Knowledge

## Animations
### `animate.enter` & `animate.leave`
Directly animate elements entering/leaving the DOM using CSS classes.
```html
<div class="fade-in" animate.enter="fade-in" animate.leave="fade-out">
  Content
</div>
```
- **Event Binding**: `(animate.enter)="onEnter($event)"`
- **Testing**: Enable animations in TestBed: `TestBed.configureTestingModule({animationsEnabled: true});`

## Component Harnesses
Abstraction layer for testing components, insulating tests from DOM structure changes.
- **Prerequisite**: install the CDK with `ng add @angular/cdk`. Built-in Material harnesses such as `MatButtonHarness` come from `@angular/material/button/testing` and need `@angular/material` (`ng add @angular/material`).

### Usage
```typescript
const loader = TestbedHarnessEnvironment.loader(fixture);
const button = await loader.getHarness(MatButtonHarness);
await button.click();
```

### Creation
Extend `ComponentHarness` and define `hostSelector`.
```typescript
export class MyComponentHarness extends ComponentHarness {
  static hostSelector = 'my-component';
  
  protected getTitleElement = this.locatorFor('h1');
  
  async getTitle(): Promise<string> {
    const title = await this.getTitleElement();
    return title.text();
  }
}
```

### Environments
- **Unit Tests**: `TestbedHarnessEnvironment` (from `@angular/cdk/testing/testbed`)
- **E2E**: `SeleniumWebDriverHarnessEnvironment` (from `@angular/cdk/testing/selenium-webdriver`), which takes a WebDriver client from the `selenium-webdriver` package; install that package separately, the CDK does not bundle it.
