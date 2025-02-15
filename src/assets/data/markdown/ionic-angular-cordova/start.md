# Lab: Generate the Application

In this lab, you will:

- Clone a repo to get started
- Build and run the starter application

## Clone the Ionic Weather Starter Template

For this training we will start with a template application that is based on the tabs starter but has all of the pages renamed to match the page structure of the weather application.

```bash
git clone https://github.com/ionic-team/ionic-weather-starter.git ionic-weather
cd ionic-weather
npm i
ionic serve
```

## Side Note: `ionic serve` vs. `npm start`

You may be used to using `npm start` to start an application. That works, but it is different. The application is for all intents and purposes an Angular application and was generated using the standard Angular schematics with some extra Ionic spices, so all of the base Angular CLI application scripts are there.

In a nutshell:

- `npm start` uses the Angular CLI directly calling `ng serve` without any options, which always tries to use port 4200
- `ionic serve` finds the first unused port >= 8100 and passes that to the Angular CLI with some other options

In either case, the Angular CLI does all the heavy lifting, so use whichever command you want to use. Personally, I use `npm start` because it take less typing and thought on my part.

## Using the CLI

Note that you could have also used the Ionic CLI to generate your application:

- `ionic start` - this will ask for information like the application's name, desired framework, and starter template.
- `ionic start my-cool-app tabs --type=angular --cordova` - this will create an `@ionic/angular` app called `my-cool-app` using the `tabs` starter template and will provide minimal Cordova scaffolding.

The reason we do not do this for this training is that you would then have to rename the files and paths yourself, which doesn't really teach you anything... 😀

## Conclusion

Congratulations on creating your `@ionic/angular` application. Next we will explore unit tests and discuss how they can help us in our development lifecycle.
