---
layout: post
title: Angular 2 Service for AJAX with Cache Management
author: ravinder_payal
---

Author:Ravinder Payal 
As we know Angular 2 sites don’t refresh and a user may revisit the same page, or want to see the same results again in a single visit to our website, so caching of AJAX request may provide great deal of fast and fluid user experience by showing results instantly.

So, here we go to create an Ajax Service for Ajax with proper cache management, which even works with parallel requests (by pausing one request and loading data with single request, and then sharing the data, this is usable when two components need same data to show something delicious) also.

For those who just want a working solution/class, it’s here.
Now start breaking the methods of class, and explaining different methods used in Ajax Service Class

```
private loadPostCache(link:string){
 if(!this.loading[link]){
           this.loading[link]=true;
           this.links[link].forEach(a=>this.dataObserver[a].next(false));
           this.http.get(link)
               .map(this.setValue)
               .catch(this.handleError).subscribe(
               values => {
                   this.data[link] = values;
                   this.links[link].forEach(a=>this.dataObserver[a].next(false));
                   delete this.loading[link];
               },
               error => {
                   delete this.loading[link];
               }
           );
       }
}
```
loadPostCache is private function and called internally if cache for specific link/url doesn’t exist. It uses http class provided by Angular 2 itself for core AJAX functionality. The function have reference to observer of all subscribers through this.dataObserver containing reference to Observable of every new observer creating by exposed function (to know keep reading). this.links[link] is another Object map containing Link to Observer Maping, this is for solving concurrent request problems, and allows to share data, with all the functions which require it, with single AJAX request.

Next is
```
postCache(link:string): Observable<Object>{

     return Observable.create(observer=> {
         if(this.data.hasOwnProperty(link)){
             observer.next(this.data[link]);
         }
         else{
             _observable=Observable.create(_observer=>{
                 this.counter=this.counter+1;
                 this.dataObserver[this.counter]=_observer;
                 this.links.hasOwnProperty(link)?this.links[link].push(this.counter):(this.links[link]=[this.counter]);
                 _observer.next(false);
             });
             this.loadPostCache(link);
             _observable.subscribe(status=>{
                 if(status){
                     observer.next(this.data[link]);
                 }
                 }
             );
         }
        });
    }
```
It seems quite complex, but it’s not. It return an observable to its caller and itself check if cache is present or not for requested link/url. If not present, it creates an observable and stores its observer reference in above discussed array, and call private function, loadPostCache, and successively subscribes just created Observable - _observable. Now the _observable waits for status to become true. `status` is set true by loadPostCache by calling next method of _observable’s observer from inside itself on completion of AJAX loading.

Usage:-

If you want to use this class in a component, you can inject it directly in that component, but first you have to register as provider in app.module.ts @NgModule, like this:-
```
app.module.ts

import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { HttpModule,JsonpModule  } from '@angular/http';/***required for AjaxService as it depends on functions provided by these two classes****/

/*
+
+
+
Other imports
+
+
*/


@NgModule({
  declarations: [
      /* --your declarations--- */
  ],
  imports: [
      /* --your imports--- */,HttpModule,JsonpModule
  ],
  bootstrap: [AppComponent],
  providers: [/*---other providers---*/, AjaxService]
});
export class AppModule {

}
```
In components and services you can use this like:-
```
import { Component } from '@angular/core';
import {AjaxService} from './service/ajax.service';

@Component({
    selector: 'home',
    templateUrl: './html/home.component.html',
    styleUrls: ['./css/home.component.css'],
})
export class HomeComponent {
    constructor(AjaxService:AjaxService){
        AjaxService.postCache("/api/home/articles").subscribe(values=>{console.log(values);this.articles=values;});
    }

    articles={1:[{data:[{title:"first",sort_text:"description"},{title:"second",sort_text:"description"}],type:"Open Source Works"}]};
}
```
Now, whenever user revisits home (without refreshing or F5), you can use cache and quickly present something delicious.

Thanks, Fire your questions here mail[-at-]ravinderpayal.com
