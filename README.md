# Spartacus Training

Welcome to the Spartacus Training Repository!

This repository is dedicated to providing comprehensive training materials and resources for Spartacus, a lean, Angular-based JavaScript storefront for SAP Commerce Cloud.

## Table of Content

- [Setup the Environment](#setup-the-environment)
  - [Versions](#versions)
  - [Hybris Environment Setup Guide](#hybris-environment-setup-guide)
  - [Angular Environment Setup Guide](#angular-environment-setup-guide)
- [Exercises](#exercises)
  - [Customizing an existing Spartacus Component](./resources/docs//exercises/01-customizing-an-existing-spartacus-component.md)

## Setup the Environment

### Versions

- **Hybris version**: *2205*

- **Angular CLI**: *15.2.10*
  - **Minimum required version**: *15.2.4*
  - **Recommended version**: The most recent *15.x*
  - **Unsupported version**: Any version *16* or higher

- **Node**: *18.13.0*
  - **Required version**: *16.13.0* or a newer *16.x* version, or else version *18.10.0* or a newer *18.x*
  - **Version supported by Angular 15 but no longer by SAP Commerce Cloud**: *14.20* and newer *14.x* versions

> [!TIP]
> Is strongly recommended to use [*Node Version Manager*](https://github.com/nvm-sh/nvm)

- **Package Manage (npm)**: *8.19.3*
  - **Minimum required version**: *8.0* or newer

- **Spartacus libraries**: 6.5.0 (We will build them during the setup)

### Hybris environment setup guide

For this training it's necessary to have an Hybris backend running to support our Spartacus frontend. For the sake of simplicity you can find an already prepared project in the '*develop*' branch. We will setup that one first.

1. Clone the repository and switch to the '*develop*' branch.
2. Extract the appropriate hybris version (2205) in the root of the project. Merge files if required.
3. Open your command prompt in the root folder and execute the `./setup_environment.sh` script. This will complete the setup for you. The process will take some time, so you can start building your Angular environment.
4. Once finished you can check if it's working properly accesing `http://localhost:9002`.

### Angular environment setup guide

1. Open a new command prompt in the root of the project and create an Angular Application using the next command:

    ```sh
      ng new mystore --style=scss --routing=false
    ```

2. Build the Spartacus libraries using the open-source code on Github and add them to the application. For more information see [Building Spartacus libraries](./resources/docs/building-spartacus-libraries.md).
3. Install the dependencies needed by your Composable Storefront app with the following command:

    ```sh
      npm install
    ```

4. Open the file `src\app\spartacus\spartacus-configuration.module.ts`, and make sure to add 'powertools-spa' to baseSite

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

5. Start your Angular frontend Application using `npm start`. You can check if it is working properly accesing `http://localhost:4200/electronics-spa/en/USD/`. Make sure your hybris backend application is running on the same time.

## Exercises

In this training, you will complete some exercises to gain practical knowledge of Spartacus. Each exercise has its own guide to follow, and you can find the resolved solutions in different branches of this repository. If you encounter difficulties, feel free to compare your code with the provided solution, but remember that if you use different versions, the code might not work as expected.

1. [Customizing an existing Spartacus Component](./resources/docs/exercises/01-customizing-an-existing-spartacus-component.md)
2. TODO
3. TODO
4. TODO
