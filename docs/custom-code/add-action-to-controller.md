---
id: add-action-to-controller
title: How to add an action to a REST API controller
sidebar_label: Add REST API Action
slug: /custom-code/controller-action
---

# How to add an action to a REST API controller

## General

In this example, you will see how to add an action to a REST API controller.

The _entity_.controller.ts file is generated only once by Amplication, and you can freely customize it. Amplication will never override this file.

You can use this file to add new actions (endpoints) or override existing actions that are inherited from _entity_.controller.base

## Example

The example will demonstrate how to get the parameters from the request and call a service to execute the operation.

It will also demonstrate how to check the user's permissions, how to add Swagger UI documentation, and how to log the request.

### Adding a new endpoint to user.controller.ts

1. Open the file **user.controller.ts**. The file is located in **./server/src/user/user.controller.ts**.

Initially, the files should look like this

```typescript
import * as common from "@nestjs/common";
import * as swagger from "@nestjs/swagger";
import * as nestAccessControl from "nest-access-control";
import { UserService } from "./user.service";
import { UserControllerBase } from "./base/user.controller.base";

@swagger.ApiBasicAuth()
@swagger.ApiTags("users")
@common.Controller("users")
export class UserController extends UserControllerBase {
  constructor(
    protected readonly service: UserService,
    @nestAccessControl.InjectRolesBuilder()
    protected readonly rolesBuilder: nestAccessControl.RolesBuilder
  ) {
    super(service, rolesBuilder);
  }
}
```

2. Add the following code at the bottom of the class.

```typescript
  @common.UseInterceptors(nestMorgan.MorganInterceptor("combined"))
  @common.UseGuards(basicAuthGuard.BasicAuthGuard, nestAccessControl.ACGuard)
  @common.Patch("/:id/password")
  @nestAccessControl.UseRoles({
    resource: "User",
    action: "update",
    possession: "own",
  })
  @swagger.ApiOkResponse({ type: User })
  @swagger.ApiNotFoundResponse({ type: errors.NotFoundException })
  @swagger.ApiForbiddenResponse({ type: errors.ForbiddenException })
  async resetPassword(
    @common.Param() params: UserWhereUniqueInput,
    @nestAccessControl.UserRoles() userRoles: string[]
  ): Promise<User | null> {
    const permission = this.rolesBuilder.permission({
      role: userRoles,
      action: "update",
      possession: "own",
      resource: "User",
    });
    const result = await this.service.restPassword({
      where: params,
    });
    if (result === null) {
      throw new errors.NotFoundException(
        `No resource was found for ${JSON.stringify(params)}`
      );
    }
    return permission.filter(result);
  }
```

The above code gets a user ID from the request, checks for the user permissions, and calls the user service to reset the password.

#### line-by-line instructions

Follow this line-by-line explanation to learn more about the code you used:

This decorator instructs morgan to log every request to this endpoint. This line is optional.

```typescript
@common.UseInterceptors(nestMorgan.MorganInterceptor("combined"))
```

This decorator instructs nestJS to guard this endpoint and prevent anonymous access.

```typescript
  @common.UseGuards(basicAuthGuard.BasicAuthGuard, nestAccessControl.ACGuard)
```

This decorator sets the route for the endpoint.

```typescript
  @common.Patch("/:id/password")
```

This decorator Uses nestJS Access Control to enforce access permissions based on the user's role permissions. This example validates that the current user can update user records.

```typescript
  @nestAccessControl.UseRoles({
    resource: "User",
    action: "update",
    possession: "own",
  })
```

These 3 decorators provide information for Swagger UI documentation

```typescript
  @swagger.ApiOkResponse({ type: User })
  @swagger.ApiNotFoundResponse({ type: errors.NotFoundException })
  @swagger.ApiForbiddenResponse({ type: errors.ForbiddenException })
```

Create a function called **resetPassword** with parameter of type **UserWhereUniqueInput** and return type **User | null**.

```typescript
async resetPassword(
    @common.Param() params: UserWhereUniqueInput,
    @nestAccessControl.UserRoles() userRoles: string[]
  ): Promise<User | null> {
```

This line creates a parameter named **userRoles** and extract its value from the current context using nestAccessControl.
**UserWhereUniqueInput** and return type **User | null**.

```typescript
    @nestAccessControl.UserRoles() userRoles: string[]
```

Create a permission object to be used later for result filtering based on the user permissions.

```typescript
const permission = this.rolesBuilder.permission({
  role: userRoles,
  action: "update",
  possession: "own",
  resource: "User",
});
```

Call the user service to execute the resetPassword, then check and filter the results before returning them to the client.

```typescript
const result = await this.service.resetPassword({
  where: params,
});
if (result === null) {
  throw new errors.NotFoundException(
    `No resource was found for ${JSON.stringify(params)}`
  );
}
return permission.filter(result);
```

## Check your changes

You are ready to check your changes. Just save all changes and restart your server.
Navigate to http://localhost:3000/api/ to see and execute the new API endpoint.

:::tip
You can run your server in watch mode so it automatically restarts every time a file in the server code is changed.
Instead of using **npm start** you should use this command

```
nest start --debug --watch
```

:::
