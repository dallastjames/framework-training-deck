# Lab: The Base Application

This training starts with an Ionic Framework application that uses standard `email/password` with a remote hosted server for authentication. This is a common paradigm used in web applications. In this lab, we will add SSO with Auth0 capability for authentication.

## Getting Started

These instrctions assume that you have a reasonable development environment set up on your machine including `git`, `node`, `npm`, and `Android Studio`. If your are using a Mac and want to build for iOS, you should also have `Xcode`, the Xcode commandline tools, and `cocoapods`.

To get started, perform the following actions within a working folder:

- `git clone https://github.com/ionic-team/ac-training-starter.git`
- `cd ac-training-starter`
- `npm i`
- `npm run build`
- `npx cap update` - this may take a while
- `npm run build`
- `npm start` - to run in the browser

To build for installation on a device, use `npx cap open android` or `npx cap open ios`. This will open the project in the appropriate IDE. From there you can build the native application and install it on your device.

## General Architecture

### Services

Two services are related to the authentication workflow. The `AuthenticationService` handles the API calls that perform login and logout actions. The `IdentiyService` handles the identity of the currently logged in user.

#### AuthenticationService

The `AuthenticationService` handles the POSTs to the login and logout endpoints. If these operations are successful it registers this fact with the `IdentityService`. This is the first of two services used in this training, so let's look at it a bit more in depth.

##### Construction

This service is registered with the dependency injection engine such that it is available to the whole application. This service exposes various methods that can be called to perform authentication actions. Currently, the supported actions include `login()` and `logout()`. 

##### `login()` - Attempt to login with email and password

Makes a call to the API that will attempt to login the user using `email` and `password` credentials. If successful, the response is unpacked and we save the user and token information into our identityService where it can be referenced by other parts of the application when needed.

```TypeScript
login(email: string, password: string): Observable<boolean> {
    return this.http
      .post<{ success: boolean; user: User; token: string }>(
        `${environment.dataService}/login`,
        {
          username: email,
          password
        }
      )
      .pipe(flatMap(r => from(this.unpackResponse(r))));
}

private async unpackResponse(r: any): Promise<boolean> {
    if (r.success) {
      await this.identity.set(r.user, r.token);
    }
    return r.success;
}
```

##### `logout()` - Logout the user

Makes a call to the API that will logout the user and invalidate the current token. After that call is completed, it will make a call to the identityService to remove the stored token and user data.

```TypeScript
logout(): Observable<any> {
    return this.http.post(`${environment.dataService}/logout`, {}).pipe(
      flatMap(() =>
        from((async () => {
          this.identity.remove();
        }
      )()))
    );
}
```

#### IdentityService

The `IdentityService` defines the identity of the currently logged in user including the authentication token associated with the user. This service also persists the token so it is available between sessions. This is the second service used in this training, so let's examine it feature by feature.

##### Construction

This service is registered with the dependency injection engine such that it is availale to the whole application. All of the data controlled by this service is private. Consumers must interact with the data via the public methods. A `changed` subject is created so other parts of the application can know when the user has changed, allowing then to requery data as needed.

```TypeScript
@Injectable({
  providedIn: 'root'
})
export class IdentityService {
  private tokenKey = 'auth-token';
  private token: string;
  private user: User;

  changed: Subject<User>;

  constructor(private http: HttpClient, private storage: Storage) {
    this.changed = new Subject();
  }
  ...
}
```

##### `set()` - Set the Current User and Token

The `set()` method takes a `User` object and a token. The `User` object is cached locally and the token is stored in a local storage mechanism. The `changed` subject is also fired. This method should be called as part of the login workflow.

```TypeScript
  async set(user: User, token: string): Promise<void> {
    this.user = user;
    await this.setToken(token);
    this.changed.next(this.user);
  }
```

##### `get()` - Get the Current User

Our API has a `users/current` endpoint that returns the `User` that is assciated with whichever authentication token is sent in the request. Our application will then do one of the following with any `get()` call:

- A `User` will already be cached (either from a `set()` call or a prior `get()` call), that user will be returned
- A `User` is not cached so we make an API call to get the user, in which case:
  - An HTTP interceptor applies the currently stored token
  - The call returns the user associated with that token and the user is cached
  - If a token does not exist or is invalid, a 401 is generated by the API

```TypeScript
  get(): Observable<User> {
    if (!this.user) {
      return this.http
        .get<User>(`${environment.dataService}/users/current`)
        .pipe(tap(u => (this.user = u)));
    } else {
      return of(this.user);
    }
  }
```

This allows our application to retrieve information about the currently logged in user after a restart. This method could be called as part of the bootstrap workflow or at any other time that data about the currently logged in user is required.

##### `getToken()` - Get the Current Token

The `getToken()` method returns the currently set token. If no token is currently set, it attempts to retrieve a token from a local storage mechanism.

```TypeScript
  async getToken(): Promise<string> {
    if (!this.token) {
      await this.storage.ready();
      this.token = await this.storage.get(this.tokenKey);
    }
    return this.token;
  }
```

This method is most commonly used whenever an HTTP call is made so the token can be added to the headers of the call. In this application, the logic that performs that action has been abstracted into an HTTP interceptor.

##### `remove()` - Remove the User and the Token

The user and the token are removed from memory and from the local storage mechanism.

```TypeScript
  async remove(): Promise<void> {
    this.user = undefined;
    await this.setToken('');
    this.changed.next(this.user);
  }
```

This method should be called as part of the login workflow.



### HTTP Interceptors

Two HTTP interceptors are used by the authentication workflow. One interceptor adds the authentication token to outgoing requests that require a token. The other interceptor redirects the application to the login page when requests fail with a 401 error.

#### AuthInterceptor

This interceptor adds the authentication token to the outgoing requests that require a token. For this application, that is every request that is not itself the `login` request.

#### UnauthInterceptor

This interceptor handles 401 errors, which are returned if the user is not logged in. It handles this situation by redirecting the application to the login page.

### Application Workflow

#### Startup

Upon starting up, the application attempts to load the `TeaCategoriesPage`. Three scenarios are possible at this point:

- A valid token has been stored from a previous session
- A token has been stored from a previous session but is no longer valid
- A token has not been stored from a previous session

In the first scenario, the `TeaCategoriesPage` successfully loads the tea categories and the use can continue using the application. In the last two scenarios, the API will return a 401 error, the `UnauthInterceptor will kick in, and the user will be redirected to the login page.
