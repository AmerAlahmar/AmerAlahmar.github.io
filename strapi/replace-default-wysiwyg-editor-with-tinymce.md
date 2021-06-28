---
title: Replace Default Wysiwyg Editor With Tinymce Editor
description: Learn in this guide how you can replace the Strapi's default Wysiwyg editor with Tinymce editor
---

# Replace Default Wysiwyg Editor With Tinymce Editor

In this guide we will see how you can replace the default Wysiwyg editor (Draftjs) in Strapi with Tinymce editor.

## Introduction

This tutorial is heavily based on [this guide](https://strapi.io/documentation/developer-docs/latest/guides/registering-a-field-in-admin.html#setup) from the strapi documentation. The idea here is to create a new filed that will be modified to use Tinymce as it's editor, but before we start, there is a few things that we should know:

- Tinymce is **NOT** a Markdown editor, it's an Html editor.
This means that the value taken from the field could contain Html tags like: `<p>Text</p>`, `<img src="..." />` and even `<table>...</table>`. Therefor you should be aware of the potential [security issues](https://www.tiny.cloud/docs-3x/extras/TinyMCE3x@Security/) and how to overcome them.  
- In order for Tinymce to work, you will need to obtain an API Key by creating an account at [Tinymce](https://www.tiny.cloud/auth/signup/) (the core editor is free :heart_eyes:)
- If you are new to strapi, make sure to take a look at this [Quick Start Guide](https://strapi.io/documentation/developer-docs/latest/getting-started/quick-start.html).

Now that we are ready, let's get our hands dirty.

## Setup

##### 1. Create a new project:

First, We will create a new project, I am calling it `my-app` you can call it whatever you like.
The `--quickstart` option will tell strapi to create a basic project with default configurations and without templates, this is just to make the process easier and to avoid any complications.

:::: tabs

::: tab yarn

```
yarn create strapi-app my-app --quickstart
```

:::

::: tab npx

```
npx create-strapi-app my-app --quickstart
```

:::

::::

After running the command, a new browser tab should be opened for you to create a new adminstration account. If it didn't, head to [localhost:1337/admin](localhost:1337/admin) and fill all the necessarily information.


##### 2. Generate a plugin:
   
Now we want to generate a new [strapi plugin](https://strapi.io/documentation/developer-docs/latest/development/plugin-customization.html), but let's first stop Strapi by pressing `Ctrl+C` or `Command+C` and `cd` into the project directory

```
# Make sure to replace "my-app" with your project name
cd my-app 
```
We will call our plugin `wysiwyg` so we should run:

:::: tabs

::: tab yarn

```
yarn strapi generate:plugin wysiwyg
```

:::

::: tab npm

```
npm run strapi generate:plugin wysiwyg
```

:::

::: tab strapi

```
strapi generate:plugin wysiwyg
```

:::

::::


##### 3. Install the needed dependencies:
To be able to use Tinymce, we will need to install its library, and because Strapi is using React, we will install Tinymce library for react `@tinymce/tinymce-react`.

**But first,** let's `cd` into the new created plugin and only then install it there:

```
cd plugins/wysiwyg
```

And then,

:::: tabs

::: tab yarn

```
yarn add @tinymce/tinymce-react
```

:::

::: tab npm

```
npm install @tinymce/tinymce-react
```

:::

::::

##### 4. Create the plugin:

In [step 2](#2-generate-a-plugin), we generated the necessarily files for any plugin. Now we need to make it ours by creating the a few files to tell Strapi how to deal with this plugin.

first we will create the necessarily directories and files (React Components), then we will write into it.

To create the directories and files (make sure you are inside the plugin directory (`.../<your app name>/plugins/wysiwyg`):
```
cd admin/src/

#This will create .../MediaLib/index.js
mkdir -p components/MediaLib/; touch components/MediaLib/index.js

#This will create .../Wysiwyg/index.js
mkdir -p components/Wysiwyg/; touch components/Wysiwyg/index.js

#This will create .../Tinymce/index.js
mkdir -p components/Tinymce/; touch components/Tinymce/index.js
```

- ###### MediaLib/index.js
  This file will handle the insertion of media i.e. insert media (images, video...etc) to Tinymce editor.
  <br/>It's important to notice here that we are using Strapi Media Library to handle the media instead of letting Tinymce handle it, and that's perfect because we don't want to let the user (The person who is using the Editor) to insert media from somewhere else, so make sure **NOT** to allow such insertion in Tinymce settings (More on that later).
  <br/>Now using your favorite editor (I am using `nano`), open the file:
  ```
  nano ./components/MediaLib/index.js
  ```
  And paste the following code then save:
  ```js
  MEDILIB CODE
  ``` 
  <br/>

- ###### Wysiwyg/index.js
  This file will be the wrapper of Tinymce editor, it will display the labels and handle the error messages
  <br/>Agin,  using your favorite editor, open the file:
  ```
  nano ./components/Wysiwyg/index.js
  ```
  And paste the following code:
  **Note:** If you get `file not found` error around the `import TinyEditor...` Ignore it for now as we will create it in the next step.
  ```js
  WYSIWYG CODE
  ``` 
- ###### Tinymce/index.js
  This is where all the work is done, it's the file that will actually implement the editor
  **Note:** mark this file as we will visit it again to configure Tinymce.
  <br/>One more time, using your favorite editor, open the file:
  ```
  nano ./components/Tinymce/index.js
  ```
  And paste the following code:
  **Note:** Make sure to replace `API_KEY` with your actual key that you obtained from Tinymce.
  ```js
  TINYMCE CODE
  ```

##### 5. Register the field and the plugin:
Our plugin is ready and waiting, but Strapi doesn't know about it yet! So we need to register it for with Strapi and tell give it a few information about it.

<br />To do so, we will edit one last file (The file is already there, we will just change the code inside it).
<br/>Last time, using your favorite editor, open the file:
**Note:** Make sure you are still inside the plugin folder `.../<your app name>/plugins/wysiwyg`
```
nano index.js
```
Delete the existing code and add the following:
```js
REGISTRATION CODE
```

##### 6. Run Strapi:
That was boring, wasn't it? Now let's have fun and see some result! Let's run strapi :satisfied:

- First, let's get back to the project folder:
  ```
  cd ../../../../
  # After running this command I will be at .../my-app
  # Make sure you are in .../<your-project-name>
  ```
- Just in case, We will clean the .cache folder
  ```
  rm .cache/ -Rd
  ```
- Re-build Strapi: 
  
  :::: tabs

  ::: tab yarn

  ```
  yarn build
  ```

  :::

  ::: tab npm

  ```
  npm run build
  ```

  :::

  ::: tab strapi

  ```
  strapi build
  ```

  :::

  ::::

- Finally, Start Strapi with the front-end development mode `--watch-admin`:

  :::: tabs

  ::: tab yarn

  ```
  yarn develop --watch-admin
  ```

  :::

  ::: tab npm

  ```
  npm run develop -- --watch-admin
  ```

  :::

  ::: tab strapi

  ```
  strapi develop --watch-admin
  ```

  :::

  ::::

When you run the last command, it will open a new tab in the browser (if it didn't, head to [localhost:8000/admin](localhost:8000/admin)) and login with the administrator account you cerated earlier.

<<<<<<<<<<<<<PICTURE>>>>>>>>>>>>>

From the menu on the left go to `Content-Types Builder` so we can create a new content for testing.

<<<<<<<<<<<<<PICTURE>>>>>>>>>>>>>

Chose `Create new single type`
<<<<<<<<<<<<<PICTURE>>>>>>>>>>>>>

Enter display name something like `Tinymce Test`.
<<<<<<<<<<<<<PICTURE>>>>>>>>>>>>>

Chose Rich Text and give it a name like `test`.
<<<<<<<<<<<<<PICTURE>>>>>>>>>>>>>

Hit `Finish` and then from the top right hit `Save`, and wait for the server to restart.
<<<<<<<<<<<<<PICTURE>>>>>>>>>>>>>

Ok, to the moment of truth. In the left menu you will find the new created content `Tinymce Test`, press it to edit it. And hop!, there you go, Tinymce is working! Yaaay :heart_eyes: 
<<<<<<<<<<<<<PICTURE>>>>>>>>>>>>>

Hmm, something isn't quite right yet! You are probably not able to insert a new line or do pretty much anything useful! Let's stop strapi and deal with it. Press `Ctrl+C` or `Command+C` to stop it.

Remember The file we marked? In that file, we need to configure Tinymce to work for us. we need to tell Tinymce `three` important things.

from the project directory, open the file using your favorite editor:

```
nano plugins/wysiwyg/admin/src/components/Tinymce/index.js
```
And do the following changes:
- outputFormat:
  To make use the full use of Tinymce, we will tell it to deal with the input as an Html and give the out put as an Html too,
  Change: `outputFormat='text'` To: `outputFormat='html'`
- selector:
  inside `init={{}}` add: `selector: 'textarea',`
  this is to tell strapi that we are using `<textarea></textarea>` tags for input.
- plugins & toolbar:
  This is where all the fun is. again, inside `init={{}}` and after the previously added `selector`, add two things:
  - `plugins: '',` Here we will add all the features and functionalities that we want Tinymce to have.
  - `toolbar: '',` It's also for adding features, but those who are added here will appear directly in the top toolbar of Tinymce, while the ones we added earlier will appear in a drop down menu.

  **Note:** Add all the plugins you want between the single quotes `' HERE '` and separate them with single spaces, [A full list can be found here](https://www.tiny.cloud/docs/plugins/opensource/), Remember not to add any plugin tat allows users to upload the media directly to the editor.

When you are done, Re-build Strapi and start it again:
  :::: tabs

  ::: tab yarn

  ```
  yarn build
  yarn develop --watch-admin
  ```

  :::

  ::: tab npm

  ```
  npm run build
  npm run develop -- --watch-admin
  ```

  :::

  ::: tab strapi

  ```
  strapi build
  strapi develop --watch-admin
  ```

  :::

  ::::