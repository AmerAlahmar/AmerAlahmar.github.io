


---

title: Replace Default Wysiwyg Editor With Tinymce Editor

description: Learn in this guide how you can replace the Strapi's default Wysiwyg editor with Tinymce editor

---



# Replace Default Wysiwyg Editor With Tinymce Editor



In this guide, we will see how you can replace the default Wysiwyg editor (Draftjs) in Strapi with the TinyMCE editor.



## Introduction



This tutorial is heavily based on [this guide](https://strapi.io/documentation/developer-docs/latest/guides/registering-a-field-in-admin.html#setup) from the Strapi documentation. The idea here is to create a new field that will be modified to use TinyMCE as its editor, but before we start, there are a few things that we should know:



- Tinymce is **NOT** a Markdown editor, it's an Html editor.

This means that the value taken from the field could contain Html tags like: `<p>Text</p>`, `<img src="..." />` and even `<table>...</table>`. Therefor you should be aware of the potential [security issues](https://www.tiny.cloud/docs-3x/extras/TinyMCE3x@Security/) and how to overcome them.

- For TinyMCE to work, you will need to obtain an API Key by creating an account at [Tinymce](https://www.tiny.cloud/auth/signup/) (the core editor is free :heart_eyes:)

- If you are new to Strapi, make sure to take a look at this [Quick Start Guide](https://strapi.io/documentation/developer-docs/latest/getting-started/quick-start.html).



Now that we are ready, let's get our hands dirty.



## Setup



##### 1. Create a new project:



First, We will create a new project, I am calling it `my-app` you can call it whatever you like.

The `--quickstart` option will tell Strapi to create a basic project with default configurations and without templates, this is just to make the process easier and to avoid any complications.



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



After running the command, a new browser tab should be opened for you to create a new administrator account. If it didn't, head to [localhost:1337/admin](localhost:1337/admin) and fill in all the necessary information.




![Register Adminstration Account](https://github.com/AmerAlahmar/AmerAlahmar.github.io/blob/main/strapi/images/register-admin-acount.png?raw=true)







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

To be able to use TinyMCE, we will need to install its library, and because Strapi is using React, we will install the TinyMCE library for React `@tinymce/tinymce-react`.



**But first,** let's `cd` into the newly created plugin and only then install it there:



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



In [step 2](#2-generate-a-plugin), we generated the necessary files for any plugin. Now we need to make it ours by creating a few files to tell Strapi how to deal with this plugin.



first, we will create the necessary directories and files (React Components), then we will write into them.



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

This file will handle the insertion of media i.e. insert media (images, video...etc) to TinyMCE editor.

<br/>It's important to notice here that we are using Strapi Media Library to handle the media instead of letting Tinymce handle it, and that's perfect because we don't want to let the user (The person who is using the Editor) to insert media from somewhere else, so make sure **NOT** to allow such insertion in Tinymce settings (More on that later).

<br/>Now using your favorite editor (I am using `nano`), open the file:

 ```

 nano ./components/MediaLib/index.js

 ```

And paste the following code then save:

 ```js


 import React, { useEffect, useState } from 'react';

 import { useStrapi, prefixFileUrlWithBackendUrl } from 'strapi-helper-plugin';

 import PropTypes from 'prop-types';



 const MediaLib = ({ isOpen, onChange, onToggle }) => {

 const {

 strapi: {

 componentApi: { getComponent },

 },

 } = useStrapi();

 const [data, setData] = useState(null);

 const [isDisplayed, setIsDisplayed] = useState(false);



 useEffect(() => {

 if (isOpen) {

 setIsDisplayed(true);

 }

 }, [isOpen]);



 const Component = getComponent('media-library').Component;



 const handleInputChange = data => {

 if (data) {

 const { url } = data;



 setData({ ...data, url: prefixFileUrlWithBackendUrl(url) });

 }

 };



 const handleClosed = () => {

 if (data) {

 onChange(data);

 }



 setData(null);

 setIsDisplayed(false);

 };



 if (Component && isDisplayed) {

 return (

 <Component

 allowedTypes={['images', 'videos', 'files']}

 isOpen={isOpen}

 multiple={false}

 noNavigation

 onClosed={handleClosed}

 onInputMediaChange={handleInputChange}

 onToggle={onToggle}

 />

 );

 }



 return null;

 };



 MediaLib.defaultProps = {

 isOpen: false,

 onChange: () => {},

 onToggle: () => {},

 };



 MediaLib.propTypes = {

 isOpen: PropTypes.bool,

 onChange: PropTypes.func,

 onToggle: PropTypes.func,

 };



 export default MediaLib;


 ``` 

<br/>



- ###### Wysiwyg/index.js

This file will be the wrapper of Tinymce editor, it will display the labels and handle the error messages

<br/>Agin, using your favorite editor, open the file:

 ```

 nano ./components/Wysiwyg/index.js

 ```

And paste the following code:

**Note:** If you get `file not found` error around the `import TinyEditor...` Ignore it for now as we will create it in the next step.

 ```js


 import React, { useState } from 'react';

 import PropTypes from 'prop-types';

 import { isEmpty } from 'lodash';

 import { Button } from '@buffetjs/core';

 import { Label, InputDescription, InputErrors } from 'strapi-helper-plugin';

 import MediaLib from '../MediaLib';

 import TinyEditor from '../Tinymce';



 const Wysiwyg = ({

 inputDescription,

 errors,

 label,

 name,

 noErrorsDescription,

 onChange,

 value,

 }) => {

 const [isOpen, setIsOpen] = useState(false);

 let spacer = !isEmpty(inputDescription) ? <div style={{ height: '.4rem' }} /> : <div />;



 if (!noErrorsDescription && !isEmpty(errors)) {

 spacer = <div />;

 }



 const handleChange = data => {

 if (data.mime.includes('image')) {

 const imgTag = `<p><img src="${data.url}" caption="${data.caption}" alt="${data.alternativeText}"></img></p>`;

 const newValue = value ? `${value}${imgTag}` : imgTag;



 onChange({ target: { name, value: newValue } });

 }



 // Handle videos and other types of files by adding some code

 };



 const handleToggle = () => setIsOpen(prev => !prev);



 return (

 <div

 style={{

 marginBottom: '1.6rem',

 fontSize: '1.3rem',

 fontFamily: 'Lato',

 }}

 >

 <Label htmlFor={name} message={label} style={{ marginBottom: 10 }} />

 <div>

 <Button color="primary" onClick={handleToggle}>

 MediaLib

 </Button>

 </div>

 <TinyEditor name={name} onChange={onChange} value={value} />

 <InputDescription

 message={inputDescription}

 style={!isEmpty(inputDescription) ? { marginTop: '1.4rem' } : {}}

 />

 <InputErrors errors={(!noErrorsDescription && errors) || []} name={name} />

 {spacer}

 <MediaLib onToggle={handleToggle} isOpen={isOpen} onChange={handleChange} />

 </div>

 );

 };



 Wysiwyg.defaultProps = {

 errors: [],

 inputDescription: null,

 label: '',

 noErrorsDescription: false,

 value: '',

 };



 Wysiwyg.propTypes = {

 errors: PropTypes.array,

 inputDescription: PropTypes.oneOfType([

 PropTypes.string,

 PropTypes.func,

 PropTypes.shape({

 id: PropTypes.string,

 params: PropTypes.object,

 }),

 ]),

 label: PropTypes.oneOfType([

 PropTypes.string,

 PropTypes.func,

 PropTypes.shape({

 id: PropTypes.string,

 params: PropTypes.object,

 }),

 ]),

 name: PropTypes.string.isRequired,

 noErrorsDescription: PropTypes.bool,

 onChange: PropTypes.func.isRequired,

 value: PropTypes.string,

 };



 export default Wysiwyg;


 ``` 

- ###### Tinymce/index.js

This is where all the work is done, it's the file that will implement the editor

**Note:** mark this file as we will visit it again to configure TinyMCE.

<br/>One more time, using your favorite editor, open the file:

 ```

 nano ./components/Tinymce/index.js

 ```

And paste the following code:

**Note:** Make sure to replace `API_KEY` with your actual key that you obtained from Tinymce.

 ```js


 import React from "react";

 import PropTypes from "prop-types";

 import { Editor } from "@tinymce/tinymce-react";



 const TinyEditor = ({ onChange, name, value }) => {



 return (

 <Editor

 apiKey="API KEY"

 value={value}

 tagName={name}

 onEditorChange={(editorContent) => {

 onChange({ target: { name, value: editorContent } });

 }}

 outputFormat='text'

 init={{}}

 />

 );

 };





 TinyEditor.propTypes = {

 onChange: PropTypes.func.isRequired,

 name: PropTypes.string.isRequired,

 value: PropTypes.string,

 };



 export default TinyEditor;




 ```



##### 5. Register the field and the plugin:

Our plugin is ready and waiting, but Strapi doesn't know about it yet! So we need to register it with Strapi and tell give it some information about it.



<br />To do so, we will edit one last file (The file is already there, we will just change the code inside it).

<br/>Last time, using your favorite editor, open the file:

**Note:** Make sure you are still inside the plugin folder `.../<your app name>/plugins/wysiwyg`

```

nano index.js

```

Delete the existing code and add the following:

```js


import pluginPkg from '../../package.json';

import pluginId from './pluginId';

import Wysiwyg from './components/Wysiwyg';



export default strapi => {

 const pluginDescription = pluginPkg.strapi.description || pluginPkg.description;

 const icon = pluginPkg.strapi.icon;

 const name = pluginPkg.strapi.name;



 const plugin = {

 blockerComponent: null,

 blockerComponentProps: {},

 description: pluginDescription,

 icon,

 id: pluginId,

 injectedComponents: [],

 isReady: true,

 isRequired: pluginPkg.strapi.required || false,

 mainComponent: null,

 name,

 preventComponentRendering: false,

 trads: {},

 };



 strapi.registerField({ type: 'wysiwyg', Component: Wysiwyg });

 return strapi.registerPlugin(plugin);

};


```



##### 6. Run Strapi:

That was boring, wasn't it? Now let's have fun and see some results! Let's run strapi :satisfied:



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



When you run the last command, it will open a new tab in the browser (if it didn't, head to [localhost:8000/admin](localhost:8000/admin)) and log in with the administrator account you created earlier.




![Login Page](https://github.com/AmerAlahmar/AmerAlahmar.github.io/blob/main/strapi/images/login-page.png?raw=true)




From the menu on the left go to `Content-Types Builder` so we can create new content for testing.




![Chose Content-Types From The Left Menu](https://github.com/AmerAlahmar/AmerAlahmar.github.io/blob/main/strapi/images/goto-content-types-builder.png?raw=true)




Chose `Create new single type`





![Chose Create New Single Type](https://github.com/AmerAlahmar/AmerAlahmar.github.io/blob/main/strapi/images/chose-create-new-single-type.png?raw=true)




Enter display name something like `Tinymce Test`.




![Name it Tinymce Test](https://github.com/AmerAlahmar/AmerAlahmar.github.io/blob/main/strapi/images/enter-display-name-tinymce-test.png?raw=true)





Chose Rich Text.





![Chose Rich Text](https://github.com/AmerAlahmar/AmerAlahmar.github.io/blob/main/strapi/images/chose-rich-text.png?raw=true)








Give it a name like `Test` and hit `Finish`.

![Name it Test](https://github.com/AmerAlahmar/AmerAlahmar.github.io/blob/main/strapi/images/name-field-test-and-finish.png?raw=true)





From the top right corner hit `Save`, and wait for the server to restart



![Click Save](https://github.com/AmerAlahmar/AmerAlahmar.github.io/blob/main/strapi/images/click-save.png?raw=true)



![Click Save](https://github.com/AmerAlahmar/AmerAlahmar.github.io/blob/main/strapi/images/wait-for-server-restart.png?raw=true)






Ok, to the moment of truth. In the left menu, you will find the newly created content `Tinymce Test`, press it to edit it. And hop!, there you go, Tinymce is working! Yaaay :heart_eyes:




![Observe Tinymce Is Broken](https://github.com/AmerAlahmar/AmerAlahmar.github.io/blob/main/strapi/images/tinymce-is-broken.png?raw=true)




Hmm :confused:, something isn't quite right yet! You are probably not able to insert a new line or do pretty much anything useful! Let's stop Strapi and deal with it. Press `Ctrl+C` or `Command+C` to stop it.




Remember The file we marked? In that file, we need to configure TinyMCE to work for us. we need to tell Tinymce `three` important things.



from the project directory, open the file using your favorite editor:



```

nano plugins/wysiwyg/admin/src/components/Tinymce/index.js

```

And do the following changes:

- outputFormat:

To make full use of TinyMCE, we will tell it to deal with the input as an Html and give the output as an Html too,

Change: `outputFormat='text'` To: `outputFormat='html'`

- selector:

inside `init={{}}` add: `selector: 'textarea',`

this is to tell strapi that we are using `<textarea></textarea>` tags for input.

- plugins & toolbar:

This is where all the fun is. again, inside `init={{}}` and after the previously added `selector`, add two things:

- `plugins: '',` Here we will add all the features and functionalities that we want Tinymce to have.

- `toolbar: '',` It's also for adding features, but those who are added here will appear directly in the top toolbar of Tinymce, while the ones we added earlier will appear in a drop-down menu.



**Note:** Add all the plugins you want between the single quotes `' HERE '` and separate them with single spaces, [A full list can be found here](https://www.tiny.cloud/docs/plugins/opensource/), Remember not to add any plugin that allows users to upload the media directly to the editor.



When you are done, Re-build Strapi and start it again:


:::: tabs





::: tab yarn





```

yarn build

yarn develop

```





:::





::: tab npm





```

npm run build

npm run develop

```





:::





::: tab strapi





```

strapi build

strapi develop

```





:::



::::



When Strapi is ready, go back to our `Tinymce Test` and give it another try, everything should be working fine :satisfied:.





![Observe Tinymce Is Working](https://github.com/AmerAlahmar/AmerAlahmar.github.io/blob/main/strapi/images/tinymce-is-working.png?raw=true)
