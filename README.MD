# Ionic2-auto-complete-ng5

This is a modified component with Angular5 and AOT(Ahead of Time) compiler support (no `--prod` build issues! Yay!). The original component was made by [kadoshms](https://github.com/kadoshms). Go and give his [repo](https://github.com/kadoshms/ionic2-autocomplete) a star!

This is a component based on Ionic's search-bar component, with the addition of auto-complete abillity.
This component is super simple and light-weight. Just provide the data, and let the fun begin.

This is a **free software** please feel free to contribute! :)

### Installation
```
$ npm install ionic2-auto-complete-ng5 save
```

#### Usage guide

Open `app.module.ts` and add the following import statetment:

``
import { AutoCompleteModule } from 'ionic2-auto-complete-ng5';
``

Then, add the `AutoCompleteModule` to the `imports` array:

```
@NgModule({
  declarations: [
    MyApp,
    HomePage,
    TabsPage,
    MyItem
  ],
  imports: [
    BrowserModule,
    AutoCompleteModule,
    FormsModule,
    HttpModule,
    IonicModule.forRoot(MyApp)
  ],
  ...
  ...
})
export class AppModule {}
```
Now let's import the styling file. Open `app.scss` and add the following:

`@import "../../node_modules/ionic2-auto-complete-ng5/auto-complete";`

Now, let's add the component to our app!

Add the following tag to one of your pages, in this example I am using the Homepage:

`<ion-auto-complete></ion-auto-complete>`

Now let's see what wev'e done so far by running `ionic serve`.

Now, when everything is up and running you should see a nice search-bar component. Open the **developer console** and try to type something.

Oh no! something is wrong. You probably see an excpetion similiar to :

`EXCEPTION: Error in ./AutoCompleteComponent class AutoCompleteComponent - inline template:1:21`

This is totatlly cool, for now. The exception shows up since we did not provide a **dataProvider** to the autocomplete component.

**How does it work?** So, ionic2-auto-complete is not responsible for getting the data from the server. As a developer, you should implement your own service which eventually be responsible to get the data for the component to work, as well we determing how many results to show and/or their order of display.

So there are two possibilities to provide data:

1. A simple function that returns an Array of items
2. An instance of 'AutocompleteService' (specified below)

Let's start by creating the service:

```
import {AutoCompleteService} from 'ionic2-auto-complete';
import { Http } from '@angular/http';
import {Injectable} from "@angular/core";
import 'rxjs/add/operator/map'

@Injectable()
export class CompleteTestService implements AutoCompleteService {
  labelAttribute = "name";

  constructor(private http:Http) {

  }
  getResults(keyword:string) {
    return this.http.get("https://restcountries.eu/rest/v1/name/"+keyword)
      .map(
        result =>
        {
          return result.json()
            .filter(item => item.name.toLowerCase().startsWith(keyword.toLowerCase()) )
        });
  }
}


```

By implementing an AutoCompleteService interface, you must implement two properties:

1. **labelAttribute** [string] - which is the name of the object's descriptive property (leaving it null is also an option for non-object results)
2. **getResults(keyword)** [() => any] - which is the method responsible for getting the data from server.

The **getResults** method can return one of:
- an Observable that produces an array
- a Subject (like an Observable)
- a Promise that provides an array
- directly an array of values

In the above example, we fetch countries data from the amazing https://restcountries.eu/ project, and we filter the results accordingly.

**Important!** the above example is just an example! the best practice would be to let the server to the filtering for us! Here, since I used the countries-api, that's the best I could do.

Now, we need to let ionic2-auto-complete that we want to use CompleteTestService as the data provider, edit *home.ts* and add `private completeTestService: CompleteTestService` to the constructor argument list.
Should look like that:
```
import { Component } from '@angular/core';
import { NavController } from 'ionic-angular';
import { CompleteTestService } from '../../providers/CompleteTestService';

@Component({
  selector: 'page-home',
  templateUrl: 'home.html'
})
export class HomePage {

  constructor(public navCtrl: NavController, public completeTestService: CompleteTestService) {

  }

}

```

Than, in *home.html* modify `<ion-auto-complete>`:
```
<ion-auto-complete [dataProvider]="completeTestService"></ion-auto-complete>
```

Now, everything should be up and ready :)


----------------------------------------------------------------------------

### Use auto-complete in Angular FormGroup ###

#### Use labelAttribute as both label and form value (default behavior) ####

By default, if your **dataProvider** provides an array of objects, the `labelAttribute` property is used to take the good field of each object to display in the suggestion list. For backward compatibility, if nothing is specified, this attribute is also used to grab the value used in the form.

The page should look like this:

```
import { Component } from '@angular/core';
import { NavController } from 'ionic-angular';
import { CompleteTestService } from '../../providers/CompleteTestService';
import { FormGroup, Validators, FormControl } from '@angular/forms'


@Component({
  selector: 'page-home',
  templateUrl: 'home.html'
})
export class HomePage {
  myForm: FormGroup

  constructor(public navCtrl: NavController, public completeTestService: CompleteTestService) {
  }

  ngOnInit(): void {
    this.myForm = new FormGroup({
      country: new FormControl('', [
        Validators.required
      ])
    })
  }

  submit(): void {
    let country = this.myForm.value.country
  }

}
```

Then, in *home.html* place the auto-complete component in the form group and add the `formControlName` attribute:
```
<form [formGroup]="myForm" (ngSubmit)="submit()" novalidate>
  <div class="ion-form-group">
    <ion-auto-complete [dataProvider]="completeTestService" formControlName="country"></ion-auto-complete>
  </div>
  <button ion-button type="submit" block>Add country</button>
</form>
```

Now when the `submit` method is called, the `country` is the selected country **name**.

**NOTE** As said above by default for backward compatibility, only the name is used as value not the country object.

#### How to use another field as form value ? ####

To indicate that you don't want the label as value but another field of the country object returned by the REST service, you can specify the attribute **formValueAttribute** on your dataProvider. For example, we want to use the country numeric code as value and still use the country name as label.

Let's update the service (juste declare `formValueAttribute` property):

```
import {AutoCompleteService} from 'ionic2-auto-complete-ng5';
import { Http } from '@angular/http';
import {Injectable} from "@angular/core";
import 'rxjs/add/operator/map'

@Injectable()
export class CompleteTestService implements AutoCompleteService {
  labelAttribute = "name";
  formValueAttribute = "numericCode"

  constructor(private http:Http) {
  }

  getResults(keyword:string) {
    return this.http.get("https://restcountries.eu/rest/v1/name/"+keyword)
      .map(
        result =>
        {
          return result.json()
            .filter(item => item.name.toLowerCase().startsWith(keyword.toLowerCase()) )
        });
  }
}
```

Now when the `submit` method is called, the `country` is the selected country **numericCode**. The name is still used as the label.

#### How to use the whole object as form value ? ####

Simply set `formValueAttribute` to empty string:
```
import {AutoCompleteService} from 'ionic2-auto-complete-ng5';
import { Http } from '@angular/http';
import {Injectable} from "@angular/core";
import 'rxjs/add/operator/map'

@Injectable()
export class CompleteTestService implements AutoCompleteService {
  labelAttribute = "name";
  formValueAttribute = ""

  constructor(private http:Http) {
  }

  getResults(keyword:string) {
    return this.http.get("https://restcountries.eu/rest/v1/name/"+keyword)
      .map(
        result =>
        {
          return result.json()
            .filter(item => item.name.toLowerCase().startsWith(keyword.toLowerCase()) )
        });
  }
}
```


----------------------------------------------------------------------------

### Styling ###

Currently for best visual result, use viewport size / fixed size (pixels) if you are interested in resizing the component:
```
ion-auto-complete {
  width: 50vw;
}
```

<!--
### How to concatenate several fields as label ? ###


The auto-complete component allows you to use templates for customize the display of each suggestion. But in many cases, the default template is good. However, you need to concatenate several fields (like firstname and lastname) to produce a full label. In that case, you can declare a method named `getItemLabel` instead of using `labelAttribute`.

For example, we want to display the country name and the population:
```
import {AutoCompleteService} from 'ionic2-auto-complete';
import { Http } from '@angular/http';
import {Injectable} from "@angular/core";
import 'rxjs/add/operator/map'

@Injectable()
export class CompleteTestService implements AutoCompleteService {
  formValueAttribute = ""

  constructor(private http:Http) {
  }

  getResults(keyword:string) {
    return this.http.get("https://restcountries.eu/rest/v1/name/"+keyword)
      .map(
        result =>
        {
          return result.json()
            .filter(item => item.name.toLowerCase().startsWith(keyword.toLowerCase()) )
        });
  }

  getItemLabel(country: any) {
    return country.name + ' (' + country.population + ')'
  }
}
```
-->

## Events ##

**itemSelected($event)** - fired when item is selected (clicked)
**itemsShown($event)** - fired when items are shown
**itemsHidden($event)** - fired when items are hidden
**ionAutoInput($event)** - fired when user inputs
**autoFocus($event)** - fired when the input is focused
**autoBlur($event)** - fired when the input is blured

## Searchbar options ##

Ionic2-auto-complete supports the regular Ionic's Searchbar options, which are set to their default values as specified in the [docs](http://ionicframework.com/docs/v2/api/components/searchbar/Searchbar/).

You can override these default values by adding the `[options]` attribute to the `<ion-auto-complete>` tag, for instance:

```
  <ion-auto-complete [dataProvider]="someProvider" [options]="{ placeholder : 'Lorem Ipsum' }"></ion-auto-complete>
```
Options include, but not limited to:
1. debounce (default is `250`)
2. autocomplete ("on" and "off")
3. type ("text", "password", "email", "number", "search", "tel", "url". Default "search".)
4. placeholder (default "Search")

## Component specific options

In addition to the searchbar options, ion-auto-complete also supports the following option attributes:

* **[template]** (TemplateRef) - custom template reference for your auto complete items (see below)
* **[showResultsFirst]** (Boolean) - for small lists it might be nicer to show all options on first tap (you might need to modify your service to handle an empty `keyword`)
* **[alwaysShowList]** (Boolean) - always show the list - defaults to false)
* **[hideListOnSelection]** (Boolean) - if allowing multiple selections, it might be nice not to dismiss the list after each selection - defaults to true)

Will set the Searchbar's placeholder to *Lorem Ipsum*

## Accessing Searchbar component ##

By using the `@ViewChild()` decorator, and the built-in `getValue()` method we can easily access the actual value in the searchbar component.
Just define a new property within the desired page, for instance (the chosen names are arbitrary):

```
  @ViewChild('searchbar')
  searchbar: AutoCompleteComponent;
```

And then, in the component tag we need to add `#searchbar`:

```
<ion-auto-complete [dataProvider]="provider" #searchbar></ion-auto-complete>
```

Available methods:

1. getValue(): `this.searchbar.getValue()` - get the string value of the selected item
2. getSelection(): `this.searchbar.getSelection()` - get the selected object
3. setFocus(): `this.searchbar.setFocus()` - focus on searchbar

## Credits ##
* [kadoshms](https://github.com/kadoshms/) for the original component and for this README
* [CoreyCole](https://github.com/CoreyCole) for the Angular 5 support
* [bushybuffalo](https://github.com/bushybuffalo) for ngModel support
