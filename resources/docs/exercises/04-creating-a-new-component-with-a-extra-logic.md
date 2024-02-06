# 3. Creating a New Component With Extra Logic

In this exercise we will create a custom component with some extra logic. For this purpose we will create a new component in both Hybris and Spartacus.

## Component Creation in Hybris

The first step will be to declare the new component type in a `-items.xml` file. For this training we will be addding it in `spartacussampledata-items.xml`.

```xml
      <itemtype code="ComponentCComponent" extends="SimpleCmsComponent"
                jaloclass="de.hybris.platform.spartacussampledata.jalo.ComponentCComponent">
          <attributes>
              <attribute qualifier="title" type="localized:java.lang.String">
                  <persistence type="property"/>
              </attribute>
              <attribute qualifier="fooCodes" type="java.lang.String">
                  <persistence type="property">
                      <columntype>
                          <value>HYBRIS.LONG_STRING</value>
                      </columntype>
                  </persistence>
              </attribute>
          </attributes>
      </itemtype>
```

For this exercise the custom CMS component will be simple, just a title and a list of codes. These codes will be used on the frontend to consume from an external API to retrieve some extra data.

We will be using the following ImPex:

```impex
$version=Staged
$contentCatalog=electronics-spaContentCatalog
$contentCV=catalogVersion(CatalogVersion.catalog(Catalog.id[default=$contentCatalog]),CatalogVersion.version[default=$version])[default=$contentCatalog:$version]
$lang=en

INSERT_UPDATE ComponentCComponent; $contentCV[unique=true]; uid[unique = true]; name; title[lang = $lang]; fooCodes;&componentRef
;; componentCTest ; Component C Test ; "This is Component C" ; charmeleon,wartortle,kakuna;componentCTest

INSERT_UPDATE ComponentTypeGroups2ComponentType; source(code)[unique=true]; target(code)[unique=true]
;wide;ComponentCComponent
;narrow; ComponentCComponent

INSERT_UPDATE ContentSlot;$contentCV[unique=true];uid[unique=true];name;active;cmsComponents(&componentRef)
;;Section2CSlot-Homepage; Content for test Section 1 Slot;true;componentCTest
```

> [!TIP]
> This impex uses the staged version by default, so you must sync the page via SmartEdit. However, if you prefer, you can switch the version to online to display the changes quickly.

## Retrieving data from external API

In this exercise we will be consuming data from [PokéAPI](https://pokeapi.co/). For that purpose we will be using a ts external library called [pokenode-ts](https://github.com/Gabb-c/pokenode-ts). You can add it to your Spartacus project using the following command:

```sh
npm install axios axios-cache-interceptor pokenode-ts
```

After that we will create a new service to manage our calls to the API:

```sh
ng g s services/poke
```

```ts
import { Injectable } from '@angular/core';
import { MainClient, Pokemon } from 'pokenode-ts';
import { Observable, combineLatest, from } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class PokeService {

  protected mainClient: MainClient;

  constructor() {
    this.mainClient = new MainClient()
   }

   getPokemonsByNameList(codes: string []): Observable<Pokemon[]>{
    return combineLatest(
      codes.map((code) => from(this.mainClient.pokemon.getPokemonByName(code)))
    );
   }
}
```

## Component Creation in Spartacus

We will create our Spartacus component using the following command:

```sh
ng g m component-c && ng g c component-c
```

We will also add it to the `app.module` using *lazy loading*:

```ts
...
import { provideConfig } from '@spartacus/core';

@NgModule({
  ...,
  providers:[
    provideConfig({
      featureModules:{
        ...,
        ComponentC:{
          module:()=> import('./component-c/component-c.module').then(m => m.ComponentCModule),
          cmsComponents:[
            'ComponentCComponent'
          ]
        }
      }
    })
  ],
  bootstrap: [AppComponent],
})
export class AppModule {}
```

For design reasons, you will also include a Presenter (or Dummy component) to render the cards. For that purpose we will create an additional standalone component with the following command:

```sh
ng g c pokemon --standalone 
```

```ts
import { Component, Input } from '@angular/core';
import { CommonModule } from '@angular/common';
import { Pokemon } from 'pokenode-ts';
@Component({
  selector: 'app-pokemon',
  standalone: true,
  imports: [CommonModule],
  templateUrl: './pokemon.component.html',
  styleUrls: ['./pokemon.component.scss']
})
export class PokemonComponent {
  @Input() pokemon?: Pokemon;
}

```

We will add the following html structure to configure our cards:

```html
<div class="card">
    <img
      class="card-img-top"
      [src]="pokemon?.sprites?.other?.home?.front_default"
      [alt]="pokemon?.base_experience"
    />
  
    <div class="card-body">
      <h5 class="card-title">
        <strong>{{ pokemon?.name | titlecase }} </strong>
        <small class="text-muted">{{ pokemon?.base_experience }} XP</small>
      </h5>
      <ul class="list-group" *ngIf="pokemon?.types">
        <h6>Types</h6>
        <li class="list-item" *ngFor="let type of pokemon?.types">
          {{ type.type.name | titlecase }}
        </li>
      </ul>
  
      <ul class="list-group" *ngIf="pokemon?.abilities">
        <h6>Abilities</h6>
        <li class="list-item" *ngFor="let ability of pokemon?.abilities">
          {{ ability.ability.name| titlecase }}
        </li>
      </ul>
    </div>
  </div>
```

We will add this component to the imports section in `component-c.module.ts`. We will also add the mapping of the component:

```ts
import { PokemonComponent } from '../pokemon/pokemon.component';

@NgModule({
  ...,
  imports: [
    CommonModule,
    PokemonComponent,
    ConfigModule.withConfig({
      cmsComponents: {
        ComponentCComponent: { // CMS Component
          component: ComponentCComponent, // Spartacus Component
        },
      },
    } as CmsConfig),
  ]
})
export class ComponentCModule { }
```

## Components and service integration

We will use the following command to create an interface to manage the data from the backend:

```sh
ng g i component-c/cms-component-c-component
```

```ts
import { CmsComponent } from "@spartacus/core";

export interface CmsComponentCComponent extends CmsComponent{
    title?: string;
    fooCodes?: string;
}
```

Now we will inject the `PokeService` and the `CmsComponentData` data stream in our component controller.

We will use the `CmsComponentData` to obtain an Observable with the data from the CMS and then use the `RxJs` *tap* operator and the `PokeService` to transform this Observable into the Observable of a Pokemon list. We will assign this last Observable to another property inside the component.

```ts
...
import { Observable } from 'rxjs';
import { CmsComponentCComponent } from './cms-component-c-component';
import { PokeService } from '../services/poke.service';
import { CmsComponentData } from '@spartacus/storefront';
import { tap } from 'rxjs/operators';
import { Pokemon } from 'pokenode-ts';
...
export class ComponentCComponent {
  public data$: Observable<CmsComponentCComponent> =
    this.componentData.data$.pipe(
      tap((data) => {
        const codes = data.fooCodes?.split(',') ?? [];
        this.pokemons$ = this.pokeService.getPokemonsByNameList(codes);
      })
    );
  public pokemons$?: Observable<Pokemon[]>;

  constructor(
    protected componentData: CmsComponentData<CmsComponentCComponent>,
    protected pokeService: PokeService
  ) {}
}
```

In the html, we will subscribe to both Observables with the async pipe and display the results. To render the cards with the Pokemon you will make use of the Presenter component.

```html
<ng-container *ngIf="data$ | async as data">
    <h1>{{data.title}}</h1>
    <hr/>
</ng-container>
<div *ngIf="pokemons$ | async as pokemons" class="row">
    <div class="col-3" *ngFor="let pokemon of pokemons">
        <app-pokemon [pokemon]="pokemon"></app-pokemon>
    </div>
</div>
```

Result:

<div align="center">
  <img src="../../media/exercise-4/4-1.png"  alt="Component result" width="400px" />
</div>

Congratulations! You have succesfully created your first component with extra logic! You can keep learning with the next [exercise](./05-cmsflexcomponent-(special-case).md).

If you encounter difficulties, feel free to compare your code with the provided [solution](https://github.com/ETuria-Labs/spartacus-training/compare/03-creating-a-new-component-with-nested-components...04-creating-a-new-component-with-extra-logic?expand=1).
