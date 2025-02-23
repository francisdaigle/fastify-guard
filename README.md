# fastify-guard
> A simple user role and scope checker plugin to protect endpoints for [Fastify](https://github.com/fastify/fastify).

[![NPM](https://nodei.co/npm/fastify-guard.png)](https://nodei.co/npm/fastify-guard/)

`fastify-guard` is designed to protect API endpoints by checking authenticated user roles and/or scopes if they met or not. `guard` is the registered Fastify decorator and can be used in anywhere.

Inspired by [express-jwt-permissions](https://github.com/MichielDeMey/express-jwt-permissions).

## Options

| Name              | Type       | Default   | Description                                                                                                          |
| ---               | ---        | ---       | ---                                                                                                                  |
| requestProperty   | string     | `user`    | The authenticated user property name that fastify-guard will search in request object                                |
| roleProperty      | string     | `role`    | The role property name that fastify-guard will search in authenticated user object                                   |
| scopeProperty     | string     | `scope`   | The scope property name that fastify-guard will search in authenticated user object                                  |
| errorHandler      | function   | undefined | Custom error handler to manipulate the response that will be returned. As fallback, default HTTP error messages will be returned. |

## API

### `guard.role(role)`

Returns a function which checks if the authenticated user has the given role(s). Multiple roles can be sent as separated parameters or in an array. If the given role(s) was not assigned to the authenticated user, the function will throw an HTTP Error (if no errorHandler provided in options otherwise the errorHandler will be invoked). The function supposed to be used in `preHandler` hook.

### `guard.hasRole(request, role)`

Returns a boolean value which indicates the authenticated user has the given role.

`request` is the Fastify request object

`role` is role name

### `guard.scope(scope)`

Returns a function which checks if the authenticated user has the given scope(s). Multiple scopes can be sent as separated parameters or in an array. If the given scope(s) was not assigned to the authenticated user, the function will throw an HTTP Error (if no errorHandler provided in options otherwise the errorHandler will be invoked). The function supposed to be used in `preHandler` hook.

### `guard.hasScope(request, scope)`

Returns a boolean value which indicates the authenticated user has the given scope.

`request` is the Fastify request object

`scope` is scope name

## Examples

```js
// get required modules
const fastify = require('fastify')()
const fastifyGuard = require('fastify-guard')

// register fastify-guard plugin
fastify.register(
  fastifyGuard,
  {
    errorHandler: (result, req, reply) => {
      return reply.send('you are not allowed to call this route')
    }
  }
)

// this route can only be called by users who has 'cto' and 'admin' roles
fastify.get(
  '/admin',
  { preHandler: [fastify.guard.role(['cto', 'admin'])] },
  (req, reply) => {
    // 'user' should already be defined in req object
    reply.send(req.user)
  }
)

// this route can only be called by users who has 'admin' or 'editor' role
fastify.get(
  '/',
  { preHandler: [fastify.guard.role('admin', 'editor')] },
  (req, reply) => {
    // 'user' should already be defined in req object
    reply.send(req.user)
  }
)

/*
http://localhost:3000 -> will print out below result if the authenticated user does not have 'admin' role

you are not allowed to call this route
*/

fastify.get(
  '/has-role',
  (req, reply) => {
    // 'user' should already be defined in req object
    reply.send(
      fastify.guard.hasRole(req, 'admin') // will return a boolean value
    )
  }
)

fastify.get(
  '/has-scope',
  (req, reply) => {
    // 'user' should already be defined in req object
    reply.send(
      fastify.guard.hasScope(req, 'profile') // will return a boolean value
    )
  }
)

// initialize the fastify server
fastify.listen(3000, () => {
  console.log('Fastify server is running on port: 3000')
})

/*
http://localhost:3000 -> will print out below result if the authenticated user does not have 'admin' role

you are not allowed to call this route
*/
```

## Installation
`npm install fastify-guard`

## Contribution
Contributions and pull requests are kindly welcomed! Please follow [Contribution Guideline](https://github.com/hsynlms/fastify-guard/blob/master/CONTRIBUTING.md) before sending PRs.

## License
This project is licensed under the terms of the [MIT license](https://github.com/hsynlms/fastify-guard/blob/master/LICENSE).
