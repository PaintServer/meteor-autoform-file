Autoform File
=============

### Description
Upload and manage files with autoForm via [`ostrio:files`](https://github.com/VeliovGroup/Meteor-Files). This package was ported from `yogiben:autoform-file` to use with [`ostrio:files`](https://github.com/VeliovGroup/Meteor-Files) instead of the already deprecated CollectionFS.

### Quick Start:

 - Install `meteor add ostrio:autoform-files`
 - Install `meteor add ostrio:files`, *if not yet installed*
 - Add this config to `simpl-schema` NPM package (depending of the language that you are using):
```javascript
SimpleSchema.setDefaultMessages({
  initialLanguage: 'en',
  messages: {
    en: {
      uploadError: '{{value}}', //File-upload
    },
  }
});
```
 - Create your Files Collection (See [`ostrio:files`](https://github.com/VeliovGroup/Meteor-Files))
```javascript
var Images = new FilesCollection({
  collectionName: 'Images',
  allowClientCode: true, // Required to let you remove uploaded file
  onBeforeUpload: function (file) {
    // Allow upload files under 10MB, and only in png/jpg/jpeg formats
    if (file.size <= 10485760 && /png|jpg|jpeg/i.test(file.ext)) {
      return true;
    } else {
      return 'Please upload image, with size equal or less than 10MB';
    }
  }
})

if (Meteor.isClient) {
  Meteor.subscribe('files.images.all');
}

if (Meteor.isServer) {
  Meteor.publish('files.images.all', function () {
    return Images.collection.find({});
  });
}
```
__Note:__ `Images` variable must be attached to *Global* scope. And has same name (*case-sensitive*) as `collectionName` option passed into `FilesCollectio#insert({collectionName: 'Images'})` method, `Images` in our case.

 - Define your schema and set the `autoform` property like in the example below
```javascript
Schemas = {};
Posts   = new Meteor.Collection('posts');
Schemas.Posts = new SimpleSchema({
  title: {
    type: String,
    max: 60
  },
  picture: {
    type: String,
    autoform: {
      afFieldInput: {
        type: 'fileUpload',
        collection: 'Images',
        uploadTemplate: 'uploadField' // <- Optional
        previewTemplate: 'uploadPreview' // <- Optional
      }
    }
  }
});

Posts.attachSchema(Schemas.Posts);
```

The `collection` property must be the same as name of your *FilesCollection* (*case-sensitive*), `Images` in our case.

Generate the form with `{{> quickform}}` or `{{#autoform}}` e.g.:

##### Insert mode:

```html
{{> quickForm collection="Posts" type="insert"}}
<!-- OR -->
{{#autoForm collection="Posts" type="insert"}}
  {{> afQuickField name="title"}}
  {{> afQuickField name="picture"}}
  <button type="submit" class="btn btn-primary">Insert</button>
{{/autoForm}}
```

##### Update mode:

```html
{{#if Template.subscriptionsReady }}
  {{> quickForm collection="Posts" type="update" doc=getPost}}
{{/if}}
<!-- OR -->
{{#if Template.subscriptionsReady }}
  {{#autoForm collection="Posts" type="update" doc=getPost}}
    {{> afQuickField name="title"}}
    {{> afQuickField name="picture"}}
    <button type="submit" class="btn btn-primary">Update</button>
  {{/autoForm}}
{{/if}}
```

Autoform should be wrapped in `{{#if Template.subscriptionsReady }}` which makes sure that template level subscription is ready. Without it the picture preview won't be shown. You can see update mode example [here](https://github.com/VeliovGroup/meteor-autoform-file/issues/9).

### Multiple images //does not support yet
If you want to use an array of images inside you have to define the autoform on on the [schema key](https://github.com/aldeed/meteor-simple-schema#schema-keys)

```javascript
Schemas.Posts = new SimpleSchema({
  title: {
    type: String,
    max: 60
  },
  pictures: {
    type: [String],
    label: 'Choose file' # optional
  },
  "pictures.$": {
    autoform: {
      afFieldInput: {
        type: 'fileUpload',
        collection: 'Images'
      }
    }
  }
})
```

### Custom file preview

Your custom file preview template data context will be:

- *file* - fileObj instance

```javascript
picture: {
  type: String,
  autoform: {
    afFieldInput: {
      type: 'fileUpload',
      collection: 'Images',
      previewTemplate: 'myFilePreview'
    }
  }
}
```

### Custom upload template

Your custom file upload template data context will be:

- *file* - FS.File instance
- *progress*
- *status*
- Other fields from [`FileUpload` instance](https://github.com/VeliovGroup/Meteor-Files/wiki/Insert-(Upload)#fileupload-methods-and-properties)

```javascript
picture: {
  type: String,
  autoform: {
    afFieldInput: {
      type: 'fileUpload',
      collection: 'Images',
      uploadTemplate: 'myFileUpload'
    }
  }
}
```

```html
<template name="myFilePreview">
  <a href="{{file.link}}">{{file.original.name}}</a>
</template>
```
