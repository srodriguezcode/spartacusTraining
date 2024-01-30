# Spartacus Training

Welcome to the Spartacus Training Repository!

This repository is dedicated to providing comprehensive training materials and resources for Spartacus, a lean, Angular-based JavaScript storefront for SAP Commerce Cloud.

## Table of Content

- [Setup the Environment](#setup-the-environment)
  - [Versions](#versions)
  - [Hybris Environment Setup Guide](#hybris-environment-setup-guide)
  - [Angular Environment Setup Guide](#angular-environment-setup-guide)
- [Exercises](#exercises)
  - [Working with components](#working-with-components)
    - [Customizing an existing Spartacus Component](./resources/docs//exercises/01-customizing-an-existing-spartacus-component.md)
    - [Creating a new component](./resources/docs/exercises/02-creating-a-new-component.md)
    - [Creating a new Component with nested Components](./resources/docs/exercises/03-creating-a-new-component-with-nested-components.md)
    - [Creating a new component with a extra logic](./resources/docs/exercises/04-creating-a-new-component-with-a-extra-logic.md)
    - [CMSFlexComponent (Special Case)](./resources/docs/exercises/05-cmsflexcomponent-(special-case).md)
    
  - [Routing](#routing)
    - [Adding Content Page](./resources/docs/routing/01-adding-content-page.md)
    - [Customizing Routes](./resources/docs/routing/02-customizing-routes.md)
    - [Using Router Link](./resources/docs/routing/03-using-router-links.md)
    - [Exercises](./resources/docs/routing/04-exercises.md)

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

### Hybris Environment Setup Guide

For this training it's necessary to have an Hybris backend with an accelerator running to support our Spartacus frontend. For the sake of simplicity you can find an already prepared project in the '*develop*' branch. We will setup that one first.

1. Clone the repository and switch to the '*develop*' branch.
2. Extract the appropriate hybris version (2205) in the root of the project. Merge files if required.
3. Open your command prompt in the root folder and execute the `./setup_environment.sh` script. This will complete the setup for you. The process will take some time, so you can start building your Angular environment.
4. Once finished start the application executing `./hybrisserver` in `hybris/bin/platform` folder. You can check if it's working properly accesing `http://localhost:9002`.

### Angular Environment Setup Guide

1. Open a new command prompt in the root of the project and create an Angular Application using the next command:

    ```sh
      ng new mystore --style=scss --routing=false
    ```

2. Build the Spartacus libraries using the open-source code on Github and add them to the application. For more information see [Building Spartacus libraries](./resources/docs/building-spartacus-libraries.md).
3. Install the dependencies needed by your Composable Storefront app with the following command:

    ```sh
      npm install
    ```

4. Open the file `src\app\spartacus\spartacus-configuration.module.ts`, and make sure it looks like this:

    ```ts
    import { NgModule } from '@angular/core';
    import { translationChunksConfig, translations } from "@spartacus/assets";
    import { FeaturesConfig, I18nConfig, OccConfig, provideConfig, SiteContextConfig } from "@spartacus/core";
    import { defaultB2bOccConfig } from "@spartacus/setup";
    import { defaultCmsContentProviders, layoutConfig, mediaConfig } from "@spartacus/storefront";

    @NgModule({
      declarations: [],
      imports: [
      ],
      providers: [provideConfig(layoutConfig), provideConfig(mediaConfig), ...defaultCmsContentProviders, provideConfig(<OccConfig>{
        backend: {
          occ: {
            baseUrl: 'https://localhost:9002'
          }
        },
        context: {
          urlParameters: ['baseSite', 'language', 'currency'],
          baseSite: ['electronics-spa'],
          currency: ['USD', 'GBP',]
        }
      }), provideConfig(<SiteContextConfig>{
        context: {},
      }), provideConfig(<I18nConfig>{
        i18n: {
          resources: translations,
          chunks: translationChunksConfig,
          fallbackLang: 'en'
        },
      }), provideConfig(<FeaturesConfig>{
        features: {
          level: '6.5'
        }
      }), provideConfig(defaultB2bOccConfig)]
    })
    export class SpartacusConfigurationModule { }

    ```
    
5. Start your Angular frontend Application using `npm start`. You can check if it is working properly accesing `http://localhost:4200/electronics-spa/en/USD/`. Make sure your hybris backend application is running on the same time.

## Exercises

In this training, you will complete some exercises to gain practical knowledge of Spartacus. Each exercise has its own guide to follow, and you can find the resolved solutions in different branches of this repository. If you encounter difficulties, feel free to compare your code with the provided solution, but remember that if you use different versions, the code might not work as expected.

### Working with components

In this section you will learn how to create and modify components in Spartacus.

The following topics will be covered in this part of the training:

1. [Customizing an existing Spartacus Component](./resources/docs/exercises/01-customizing-an-existing-spartacus-component.md)
2. [Creating a new component](./resources/docs/exercises/02-creating-a-new-component.md)
3. [Creating a new Component with nested Components](./resources/docs/exercises/03-creating-a-new-component-with-nested-components.md)
4. [Creating a new component with a extra logic](./resources/docs/exercises/04-creating-a-new-component-with-a-extra-logic.md)
5. [CMSFlexComponent (Special Case)](./resources/docs/exercises/05-cmsflexcomponent-(special-case).md)

### Routing

In this segment, we'll dive into the fundamental concept of routing within the Spartacus framework.

In this section, we will cover the following topics:

1. [Adding Content Page](./resources/docs/routing/01-adding-content-page.md)
2. [Customizing Routes](./resources/docs/routing/02-customizing-routes.md)
3. [Using Router Link](./resources/docs/routing/03-using-router-links.md)
4. [Exercises](./resources/docs/routing/04-exercises.md)
