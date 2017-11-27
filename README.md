# ngIfMedia

A flexible directive/service module for handling media types and queries in Angular 2+, inspired by [include-media](https://include-media.com/).  
  
Server rendering ([Universal](https://universal.angular.io/)) compatible.

`npm install --save ng-if-media`

```ts
import { NgIfMediaModule } from 'ng-if-media';

const mediaConfig = {
  breakpoints: {
    tablet: {
      value: '768px',
      param: 'width'
    },
    budgetHeight: {
      value: '480px',
      param: 'height'
    },
    widescreen: {
      value: '1280px',
      param: 'width'
    },
    print: {
      media: 'print'
    },
    landscape: '(orientation: landscape)'
  },
  vendorBreakpoints: ['bootstrap'],  // include 3rd party namespace
  throttle: 100
};

@NgModule({
  imports: [NgIfMediaModule.withConfig(mediaConfig)]
})
```

## Features

`ngIfMedia` allows using preconfigured breakpoints with `<, >, =` logical operators, enabling expressive and readable control over your application UI. Window `resize` updates are handled automatically during the component lifetime with a configurable throttle timer.

```html
<div *ifMedia="<tablet">I will appear below tablet width!</div>
<!-- (max-width: 767px) -->

<div *ifMedia=">=tablet and landscape">Tablets and above in landscape mode!</div>
<!-- (min-width: 768px) and (orientation: landscape) -->

<div *ifMedia="<=tablet, landscape">Tablets and below OR landscape mode!</div>
<!-- (max-width: 768px), (orientation: landscape) -->

<div *ifMedia="print">You will see this on paper IRL.</div>
<!-- (only) print -->
```

## Directive

When used as an attribute directive, `ngIfMedia` works just like `ngIf` by showing or hiding elements based on the active media query. It's therefore compatible with the [void](https://angular.io/guide/animations#the-void-state) state of native Angular 4+ animations.

```html
<nav class="desktop-nav" *ifMedia=">mobile">
  ...
</nav>

<nav class="burger-nav" *ifMedia="<=mobile">
  <ul>
    <li>About Us</li>
    <li>About You</li>
    <li>About Them</li>
  </ul>
</nav>
```

```html
<ul>
  <li *ifMedia=">tablet">
    <h1>This is a story...<h1>
    <p>of a man with many devices.</p>
    <audioPlayer play="runningman">
  </li>
  <li *ifMedia="<=tablet and >phone">
    <h1>And his struggle...</h1>
    <p>to find the smoothest scaling experience ever...</p>
  </li>
  <li *ifMedia="<=phone">
    <h1>Where every pixel...</h1>
    <p>has a standalone component.</p>
    <img src="lovecraftianhorror.jpg">
  </li>
</ul>
```

Just like in your alt schüle CSS, abstractions can be combined with the `and` keyword or comma separated and treated as independent junctions. You can also use `width` values directly or create breakpoints for other properties in the config.

```html
<span *ifMedia="<576px, >widescreen">
  I go high or I go low.
</span>

<loFiWidget *ifMedia="<=budgetHeight and landscape">
  I exist to create a great customer experience for the glorious "landscape" of budget phone users!
</loFiWidget>
```

## Service

Sometimes you need more granularity than just showing and hiding some HTML. `ngIfMedia` is also available as a service to simplify working with [matchMedia](https://developer.mozilla.org/en-US/docs/Web/API/Window/matchMedia) API and related window events using the same methodology and configuration as the directive.

```jsx
import { NgIfMediaService } from 'ng-if-media';

export class AppComponent implements OnDestroy {
  mediaContainer;

  constructor(private mediaService: NgIfMediaService) {
    this.mediaContainer = this.mediaService.register();
  }

  ngOnDestroy() {
    this.mediaContainer.deregister();
  }
}
```

### onChange

The `onChange` method provides the functional jackhammer for media specific application logic, where the media expression result is passed to the callback as an optional parameter.

```jsx
// component.html
<a routerLink="/register">{{ message }}</a>

// component.ts
const messageSmall = 'Tap to win great stuff!';
const messageBig = 'Click here to win the greatest prizes of all time in history!';
this.mediaContainer.onChange('<768px', (match) => { this.message = match ? messageSmall : messageBig });
```

The callback is executed once for every time the media expression result is flipped, allowing some advanced usage.

```jsx
// component.html
<h1>Spin to win!</h1>
<strong>x{{orientationFlipCounter}}!</strong>

// component.ts
this.mediaContainer.onChange('landscape', () => { this.orientationFlipCounter++; });
```

### when

`when` method executes its callback only once for every time the media expression evaluates to `true`.

```jsx
this.mediaContainer.when('landscape', () => {
  alert('Switching back to portrait is $2.24 monthly. - Comcast');
})
```

To make things practically simple, you can pass an object of properties to change in the current component `this` context instead.

```jsx
// component.html
<p>{{text}}</p>

// component.ts
text: string = 'Default message';
this.mediaContainer.when({
  '<=phone': { 'text': 'Text for phones' },
  '>phone and <desktop': { 'text': 'Longer text for some other devices' },
  '>=desktop': { 'text': 'Funny retro-viral message for Wizards in 8bit|4K' }
})
```

## Configuration

By default, `ngIfMedia` has no abstract configuration and you can use it freely with direct values (eg. `<=640px`) or 3rd party presets. Since project requirements can get very specific, supplying your own custom breakpoints is the most expected usecase.

You can either create flexible breakpoints that utilize `<, >, =` logical operators (with precision), specify media types and append static suffixes:

```js
breakpoints: {
  vga: {
    value: '640px',
    param: 'width',
    media: 'screen',
    suffix: '(aspect-ratio: 4/3)'
  },
  retina2: {
    value: '2dppx',
    param: 'resolution',
    precision: 0.01
  }
```

Or handle any other usecase with static string expressions (which cannot use operators):

```js
breakpoints: {
  landscape: '(orientation: landscape)',
  iPhone5: 'screen and (min-width: 320px) and (max-width: 568px) and (-webkit-min-device-pixel-ratio: 2)'
}
```

Presets are available for 3rd party methodologies (currently for [Bootstrap 4](https://v4-alpha.getbootstrap.com/layout/overview/#responsive-breakpoints)) and optionally extend the custom configuration. Resize update throttle timer is also configurable (default `100`).

```js
breakpoints: { ... },
vendorBreakpoints: ['bootstrap'],
throttle: 16.7
```


-----
Created with :muscle: in [Salsita](https://www.salsitasoft.com/)!
