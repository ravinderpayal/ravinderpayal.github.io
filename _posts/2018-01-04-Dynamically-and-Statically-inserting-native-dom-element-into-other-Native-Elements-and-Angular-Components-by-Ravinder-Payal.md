---
layout: post
title: Dynamically and statically inserting native dom element into other native elements and angular components by Ravinder Payal
author: ravinder_payal
---
<p>
First of all happy new year everyone reading out. In this short post I will show how native dom element can be inserted into already present native/angular component inside a angular component using (Renderer2)[https://angular.io/api/core/Renderer2] inside (Angular directive)[https://angular.io/api/core/Directive].
</p>

The code  is self explanatory and we need not elaborate much. So, let's dive into the code.

# For inserting element statically/once.
```typescript
import {OnInit, Directive, Input, Inject, Renderer2,ElementRef } from '@angular/core';
import { DOCUMENT} from '@angular/common';
@Directive({
  selector: '[appInsertDomElement]'
})
export class InsertDomElementDirective implements OnInit{
  @Input() element;
  constructor(private elementRef: ElementRef, private renderer: Renderer2, @Inject(DOCUMENT) private document) {}
   ngOnInit() {
     this.renderer.appendChild(this.elementRef.nativeElement, this.element);
   }
}
```

# For inserting element dynamically.
```typescript
import {Directive, Input, Inject, Renderer2,ElementRef } from '@angular/core';
import { DOCUMENT} from '@angular/common';
@Directive({
  selector: '[appInsertDomElement]'
})
export class InsertDomElementDirective{
  private lastElementRef = null;
  @Input() set element(element) {
    if (this.lastElementRef) {
      this.renderer.removeChild(this.elementRef.nativeElement, this.lastElementRef);
    }
    this.renderer.appendChild(this.elementRef.nativeElement, element);
    this.lastElementRef = element;
  };
  constructor(private elementRef: ElementRef, private renderer: Renderer2, @Inject(DOCUMENT) private document) {}
}
```

# Usage
Usage is also pretty simple

## Usage Example 1
```html
<div *ngFor="let video of subscribedVideos">
  <div appInsertDomElement [element]= "video"></div>
  <button mat-icon-button><mat-icon>pause</mat-icon></button>
  <button mat-icon-button><mat-icon>stop</mat-icon></button>
</div>
```
## Usage Example 2
```html
  <div appInsertDomElement [element]= "someElementRef"></div>
```


Thanks for visiting the blog. Wish you a productive day.
