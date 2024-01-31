## 6. Custom Services

In general, overrides in Spartacus are done through configuration in modules, which must be part of the application. Here are several service customizations, with different scopes.

### Customizing the CmsService in Spartacus.

First, let's extend the CmsService. Overrides in Spartacus are typically achieved through module configurations. Below is the process for creating a service that inherits from Spartacus' CmsService

```sh
   ng g s services/custom-cms-service/customCms
```

Extend the original service and add a message to the console in an existing method:

`custom-cms.service.ts`
```ts
import { Injectable } from '@angular/core';
import { CmsComponent, CmsService, PageContext } from '@spartacus/core';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root',
})
export class CustomCmsService extends CmsService {
  protected message: string = '<< Hello, this is Custom Cms Service >> ';

  override getComponentData<T extends CmsComponent | null>(
    uid: string,
    pageContext?: PageContext
  ): Observable<T> {
    console.log(this.message);
    return super.getComponentData(uid, pageContext);
  }
}
```

### Implementing a Global Custom Service

To replace the Out of the Box implementation of the Service with your new custom implementation, use Angular's Dependency Injection system. Here's how to override the services in your application:

`app.module.ts`
```ts
@NgModule({
  declarations: [AppComponent],
...
  providers:[
...
    {
      provide:CmsService,
      useClass: CustomCmsService
    },
  ],
  bootstrap: [AppComponent],
})
export class AppModule {}
```

Result:

<div align="center">
  <img src="../../media/exercise-6/6-1.png"  alt="Component result" width="400px" />
</div>

### Applying Custom Service in Specific Components

>[!IMPORTANT]
> Please before to continues the exercise revert the changes in `app.module`.

In cases where the new service implementation needs to be restricted to specific components, configure the component as shown below. This approach ensures that while the global application uses the default service, `ComponentBComponent` will use the custom service.

`component-b.module.ts`
```ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { ComponentBComponent } from './component-b.component';
import { CmsConfig, CmsService, ConfigModule } from '@spartacus/core';
import { PageComponentModule } from '@spartacus/storefront';
import { CustomCmsService } from '../services/custom-cms-service/custom-cms.service';

@NgModule({
  declarations: [ComponentBComponent],
  imports: [
    CommonModule,
    PageComponentModule,
    ConfigModule.withConfig({
      cmsComponents: {
        ComponentBComponent: {
          // CMS Component
          component: ComponentBComponent,
          providers: [
            {
              provide: CmsService,
              useClass: CustomCmsService,
            },
          ], // Spartacus Component
        },
      },
    } as CmsConfig),
  ],
})
export class ComponentBModule {}
```

Result:

<div align="center">
  <img src="../../media/exercise-6/6-2.png"  alt="Component result" width="600px" />
</div>

> [!NOTE]
> If you do the same with an Out of the Box component that also uses this service, you will get identical results.