# Spartacus Libraries

## Step 1. Creating a New Angular App

1. Open terminal.
2. Using the angular CLI, generate a new Angular Application with following command:

```sh
ng new mystore --style=scss --routing=false
```

3. Access the newly created `mystore` folder with the followingg command:

```sh
cd mystore
```

## Step 2. Installing Composable Storefront Libraries

1. Follow the steps to install **[Spartacus Libraries](https://github.com/SAP/spartacus/blob/release/6.0.x/docs/self-publishing-spartacus-libraries.md)**
2. In the step where you have to checkout a branch from the Spartacus repository, choose the latest release branch (such as release/6.6.x):

```bash
git checkout release/6.6.x
```

3. Check if **Verdaccio** is running at http://localhost:4873/

## Step 3. Setting your project using schematics

1. Add the Spartacus schematics package to the main Angular app

```sh
ng add @spartacus/schematics
```

2. Proceed with the installation and make sure to select all features that belong to the Organization and Product blocks as well as the Checkout B2B and Checkout Scheduled Replenishment features

> [!IMPORTANT] 
> - Composable storefront does not support B2C and B2B storefronts running together in a single storefront application.
> - If you select a feature that is for B2B storefronts, the schematics automatically add any required B2B configurations if they are missing. If you install any of the following features, your composable storefront will automatically become a B2B storefront:
>   - Organization - Administration
>   - Organization - Order Approval
>   - Product - Bulk Pricing
>   - Product Configuration - CPQ Configurator"

<img src="./media/b2b-dependencies.png" width="" height=""/>// TODO  Marcar todo B2B

## Step. 4 Installing dependencies

Install the dependencies needed by your Composable Storefront app with the following command:

```sh
npm install
```

## Step. 5 Verifying the Base URL and Other Settings
Verifying the Base URL and Other Settings Open the `src\app\spartacus\spartacus-configuration.module.ts`, and check for any changes you want to make for your setup. For example:

**baseUrl**: Point to your SAP Commerce Cloud server.
**features level**: Defines the compatibility level.
**context**: Defines the site context, such as base site, language, and currency.

```ts
...
@NgModule({
  declarations: [],
  imports: [
  ],
  providers: [provideConfig(layoutConfig), provideConfig(mediaConfig), ...defaultCmsContentProviders, provideConfig(<OccConfig>{
    backend: {
      occ: {
        baseUrl: 'https://localhost:9002',
        context: {
          urlParameters: ['baseSite', 'language', 'currency'],
          baseSite: ['electronics-spa','apparel-uk-spa','powertools-spa'],
          currency: ['USD', 'GBP',]
        },
      }
  },
})
...
]
})
```

## Step. 6 Start Your Composable Storefront App

```sh
npm start
```

Check if it is working at http://localhost:4200/electronics-spa/en/USD/