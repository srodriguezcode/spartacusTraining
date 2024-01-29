# Customizing Routes

## Table of Contents

1. [Modifying an Existing Route](#modifying-an-existing-route)
2. [How to normalize the route?](#how-to-normalize-the-route?)
3. [Creating a new Product interface](#creating-a-new-product-interface)
4. [Creating a new normalizer for categories](#creating-a-new-normalizer-for-categories)
5. [Considerations](#considerations)

## Modifying an Existing Route

### Step 1. Check That the Old Route Is Working

```http
http://localhost:4200/electronics-spa/en/USD/product/300938/Photosmart%20E317%20Digital%20Camera
```

### Step 2. Add `ConfigModule` to Your `CustomRoutingModule` And Add the New Config for Product Route

```ts
...

const STATIC_ROUTES: Routes = [
	{
		path: 'static-page',
		component: StaticPageComponent,
		canActivate: [CmsPageGuard],
		data:{ pageLabel: 'cart' }
	},
  {
    path: 'alias/cns',
    component: PageLayoutComponent,
    data: { pageLabel: '/faq' },
    canActivate: [CmsPageGuard]
  },
  {
    path: 'contact',
    component: ContactComponent,
    data: { pageLabel: '/contact' },
    canActivate: [CmsPageGuard]
  }
];

@NgModule({
  declarations: [],
  imports: [
    CommonModule,
    RouterModule.forChild(STATIC_ROUTES),
    ConfigModule.withConfig({
      routing: {
        routes: {
          product: {
            paths: [
              'electronics/cameras/:productCode'
            ]
          }
        }
      }
    } as RoutingConfig)
  ]
})
export class CustomRoutingModule { }
```

### Step 3. Now Check That the Old Route Is Not Working And the New Route Is Working

```http
http://localhost:4200/electronics-spa/en/USD/electronics/cameras/300938
```

### Step 4. Add a New Alias Above 

```ts
...
'electronics/cameras/:productCode/:name'
...
```

### Step 5. Check That Now Is Working For *Both* paths

```http
http://localhost:4200/electronics-spa/en/USD/electronics/cameras/300938
```
And

```http
http://localhost:4200/electronics-spa/en/USD/electronics/cameras/300938/Photosmart%20E317%20Digital%20Camera
```

### Step 6. Let's Add Another Alias, But This Time for *Manufacturer*

```ts
...
'electronics/cameras/:manufacturer/:productCode/:name'
...
```

> [!FAIL] 
> Now if you go and click in the product again the URL will not works because manufacturer is not provided when it's not need it

### Step 7. To Ensure the *Manufacturer* Takes Effect, Customize `OccConfig`

```ts
...
ConfigModule.withConfig({
  backend: {
	occ: {
	  endpoints: {
		productSearch:
		  // tslint:disable-next-line: max-line-length
		  'products/search?fields=products(code,manufacturer,name,summary,price(FULL),images(DEFAULT),stock(FULL),averageRating),facets,breadcrumbs,pagination(DEFAULT),sorts(DEFAULT),freeTextSearch&query=${query}',
		product: {
		  list: 'products/${productCode}?fields=code,name,manufacturer,summary,price(formattedValue),images(DEFAULT,galleryIndex)',
		},
	  },
	},
  },
} as OccConfig),
...
```

> [!CAUTION]
> `product_scopes` is not working

### Step 8. Now If You Go and Click on the Product Again the Manufacturer Will Appear

```http
http://localhost:4200/electronics-spa/en/USD/electronics/cameras/300938/HP/Photosmart%20E317%20Digital%20Camera
```

## How to Normalize the Route?

<img src="../../media/routing/route-normalize-flow.png">

### Step 1. Create the `ProductNameNormalizerService``

```sh
ng generate service ProductNameNormalizer
```

### Step 2. Make the Class a Normalizer

To make the class a normalizer, we need to implements a *Converter* interface

```ts
...

@Injectable({
  providedIn: 'root'
})
export class ProductNameNormalizerService implements Converter<Occ.Product, Product> {

  constructor() { }

  convert(source: Occ.Product, target?: Product | undefined): Product {
    throw new Error('Method not implemented.');
  }
}
```

### Step 3. Modify the *convert* Method to Normalize the Name of the Product

We need to replace all the spaces with dashes

```ts
...
convert(source: Occ.Product, target?: Product | undefined): Product {
  if (source.name && target) {
    target.name = source.name?.replace(/ /g, '-');
  }

  return target? target : {};
}
...
```

> [!WARNING]
> If you don't use the `source.name` conditional it will give you an error. Normalizers receive parts of the object such as **source** without a name

### Step 4. Provide the New Normalizer to Angular in `AppModule`

```ts
...
providers: [
  ActiveCartService,
  { provide: PRODUCT_NORMALIZER, useClass: ProductNameNormalizerService, multi: true}
],
...
```

>[!NOTE]
> `PRODUCT_NORMALIZER` is an injection token coming from *Spartacus*
> `multi` is set to true because we can have multiple providers for each model

### Step 6. Check That the Url Replaces Spaces With Dashes

**Old Url**

```http
http://localhost:4200/electronics-spa/en/USD/electronics/cameras/300938/HP/Photosmart%20E317%20Digital%20Camera
```

**New Url**

```http
http://localhost:4200/electronics-spa/en/USD/electronics/cameras/300938/HP/Photosmart-E317-Digital-Camera
```

Now, if you check the name of the product in *PDP*, it's also with dashes. To fix this we need to create a new Product interface with a new field

<img src="../../product-name-pdp.png">

## Creating a New Product Interface

### Step 1. Generate the New Interface and Extends `Product`

```sh
ng generate interface TrProduct
```

```ts
...

export interface TrProduct extends Product {
  nameForUrl?: string;
}
```

### Step 2. Modify `ProductNameNormalizer` with the New Interface

```ts
...
convert(source: Occ.Product, target?: TrProduct | undefined): TrProduct {
  if (source.name && target) {
    target.nameForUrl = source.name?.replace(/ /g, '-');
  }
    
  return target? target : {};
}
...
```

### Step 3. Change `name` to `nameForUrl` in Paths of Your `CustomRoutingModule`

```ts
...
paths: [
  'electronics/cameras/:productCode/:manufacturer/:nameForUrl',
  'electronics/cameras/:productCode/:nameForUrl',
  'electronics/cameras/:productCode'
]
...
```

> [!TIP]
> If we want to change an alias that has multiple occurrences we can use `paramsMapping``
> ```ts
> ...
> paths: [
>   'electronics/cameras/:productCode/:manufacturer/:name',
>   'electronics/cameras/:productCode/:name',
>   'electronics/cameras/:productCode',
> ],
> paramsMapping: {
>   name: 'nameForUrl'
> }
> ...

Now you can see how the *URL* has the dashes, and the name of the product in *PDP* has spaces.

> [!NOTE] Exercise 
> To be more *SEO* friendly try to make the name appears in lowercase in the url

## Creating a New normalizer for Categories

### Step 1. Generate the Normalizer

```sh
ng generate service ProductCategoryNormalizer
```

### Step 2. Implement the Converter as with `ProductNameNormalizer`

### Step 3. Add the `firstCategory` String Property to the Custom Product Interface `TrProduct`

### Step 4. Update the **convert** Method to Include the `firstCategory`

```ts
...

@Injectable({
  providedIn: 'root'
})
export class ProductCategoryNormalizerService implements Converter<Occ.Product, Product> {

  constructor() { }

  convert(source: Occ.Product, target?: TrProduct | undefined): TrProduct {
    if (source.categories?.length && target) {
      target.firstCategory = source.categories[0].name;
    }
    
    return target ?? {};
  }
}
```

### Step 5. Add the New Provider in `AppModule`

```ts
...
providers: [
...
  {
    provide: PRODUCT_NORMALIZER,
    useClass: ProductCategoryNormalizerService,
    multi: true,
  },
],
...
```

> [!NOTE]
> Notice how we also use the same token due to the multi-provider setup.
> What actually happens is that Spartacus, behind the scenes, aggregates all the providers and executes them sequentially

### Step 6. Add Another Alias that Includes `firstCategory` in `CustomRoutingModule`

```ts
...
paths: [
  'electronics/cameras/:firstCategory/:productCode/:manufacturer/:name',
  'electronics/cameras/:productCode/:manufacturer/:name',
  'electronics/cameras/:productCode/:name',
  'electronics/cameras/:productCode',
],
paramsMapping: {
  name: 'nameForUrl'
}
...
```

### Step 7. Check that the `firstCategory` now appears

```http
http://localhost:4200/electronics-spa/en/USD/electronics/cameras/Digital%20Compacts/300938/HP/photosmart-e317-digital-camera
```

> [!NOTE] 
> Try creating a new normalizer or modifying the existing one to ensure all categories appear.

## Considerations

> [!NOTE]  
> If **Early login feature** es enabled globally routes are protected by default
> Only mandatory parameters matter

> [!TIP]
> Spartacus routing configuration is located in `spartacus/projects/storefrontlib/cms-structure/routing/default-routing-config.ts`. This configuration can be overridden.

> [!WARNING]
> The order of the aliases matters
> Changing some default URL's requires modifications in the backend
> Avoid mapping `productCode` or `categoryCode` because it would result in 404 error
