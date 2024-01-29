# Exercises

## Table of Contents

1. [Exercise 01: Wishlist Page](#exercise-01---wishlist-page)
2. [Exercise 02: Routes](#exercise-02---routes)

## Exercise 01 - WishList Page

### 1. Create a new component called “wishlist”

<details>
  <summary>Solution</summary>
  <p>Generate the component with *Angular CLI</p>

```sh
ng generate component wishlist
```

</details>

### 2. Add some wishes to your wishlist in the component’s template

<details>
  <summary>Solution</summary>

```html
<p>wishlist works!</p>
<ul>
    <li>Smartphone - $500</li>
    <li>Headphones - $100</li>
    <li>Smartwatch - $300</li>
    <li>Laptop - $1200</li>
    <li>Bluetooth Speaker - $80</li>
    <li>Camera - $600</li>
</ul>
```
</details>

### 3. Configure your project so that the /my-account/wishlist route directs to this component

<details>
  <summary>Solution</summary>
  <p>Add the path to your <strong>STATIC_ROUTES</strong> in <em>CustomRoutingModule</em></p>

```ts
const STATIC_ROUTES: Routes = [
  ...,
  {
    path: 'my-account/wishlist',
    component: ContactComponent,
    data: { pageLabel: '/my-account/wishlist' },
    canActivate: [CmsPageGuard]
  }
];
```
</details>

## Exercise 02 - Routes

### 1. Configure the product details route so that it conforms to the following SEO guidelines

> [!NOTE]
> We will use the previous ProductNameNormalizerService created. If you don't have it refer to [02 Customizing Routes](./02-customizing-routes.md)

a) Contains first two category names, product code, product name, manufacturer name and contains “oldschool” and “cameras” keywords

<details>
  <summary>Solution</summary>
  <p>1. Modify your custom Product: <em>TrProduct</em> to include the second category</p>

```ts
export interface TrProduct extends Product {
  ...
    secondCategory?: string;
  ...
}
```
  <p>2. Add your new route in <em>CustomRoutingModule</em> and add oldschool and cameras</p>

```ts
  ...
  product: {
    paths: [
      'oldschool/cameras/:firstCategory/:secondCategory/:productCode/:name/:manufacturer',
  ...
```

  <p>3. Add the second category in your <em>ProductCategoryNormalizerService</em></p>

```ts
  ...
  convert(source: Occ.Product, target?: TrProduct | undefined): TrProduct {
  if (source.categories?.length && target) {
      target.firstCategory = source.categories[0].name;
      target.secondCategory = source.categories.length >= 2 ? source.categories[1].name : '';
    }

    return target ?? {};
  }
  ...
```
</details>

b) Whole URL is lowercase and replace spaces with dashes and Limit product name to 10 characters

<details>
  <summary>Solution</summary>
  <p>1. Create a new file in <em>custom-routing</em> directory that contains the prettier function</p>

 ```ts
export function prettify(str: string): string {
  return str.replace(/ /g, '-').toLowerCase();
}
```

  <p>2. Modify <em>ProductNameNormalizerService#convert</em> to prettify the route</p>

```ts
...
convert(source: Occ.Product, target?: TrProduct | undefined): TrProduct {
  if (source.name && target) {
    target.nameForUrl = prettify(source.name).substring(0, 10);
  }

  return target? target : {};
}
...
```

  <p>3. Modify <em>ProductCategoryNormalizerService#convert</em></p>

```ts
convert(source: Occ.Product, target?: TrProduct | undefined): TrProduct {
  if (source.categories?.length && target) {
    target.firstCategory = source.categories[0].name;
    target.secondCategory = source.categories.length >= 2 && source.categories[1].name ? prettify(source.categories[1].name) : '';
  }  

  return target ?? {};
}
```

</details>

Do you think that we need another normalizer for manufacturer? If you think so explain why and let's try it.

> [!IMPORTANT]
> Remember to Add fallback aliases when some parameters are unavailable
