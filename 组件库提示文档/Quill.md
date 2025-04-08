A [Quill](https://quilljs.com/) component for [React](https://facebook.github.io/react/).
### Using Deltas
You can pass a [Quill Delta](https://quilljs.com/docs/delta/), instead of an HTML string, as the `value` and `defaultValue` properties. Deltas have a number of advantages over HTML strings, so you might want use them instead. Be aware, however, that comparing Deltas for changes is more expensive than comparing HTML strings, so it might be worth to profile your usage patterns.
Note that switching `value` from an HTML string to a Delta, or vice-versa, will trigger a change, regardless of whether they represent the same document, so you might want to stick to a format and keep using it consistently throughout.
⚠️ Do not use the `delta` object you receive from the `onChange` event as `value`. This object does not contain the full document, but only the last modifications, and doing so will most likely trigger an infinite loop where the same changes are applied over and over again. Use `editor.getContents()` during the event to obtain a Delta of the full document instead. ReactQuill will prevent you from making such a mistake, however if you are absolutely sure that this is what you want, you can pass the object through `new Delta()` again to un-taint it.

```jsx
import React, { useState } from 'react';
import ReactQuill from 'react-quill';
import 'react-quill/dist/quill.snow.css';

function MyComponent() {
  const [value, setValue] = useState('');

  return <ReactQuill theme="snow" value={value} onChange={setValue} />;
}
```

### Themes
The Quill editor supports [themes](http://quilljs.com/docs/themes/). It includes a full-fledged theme, called *snow*, that is Quill's standard appearance, and a *bubble* theme that is similar to the inline editor on Medium. At the very least, the *core* theme must be included for modules like toolbars or tooltips to work.
To activate a theme, pass the name of the theme to the `theme` [prop](https://github.com/zenoamaro/react-quill#props). Pass a falsy value (eg. `null`) to use the core theme.
```
<ReactQuill theme="snow" .../>
```
Then, import the stylesheet for the themes you want to use.
This may vary depending how application is structured, directories or otherwise. For example, if you use a CSS pre-processor like SASS, you may want to import that stylesheet inside your own. These stylesheets can be found in the Quill distribution, but for convenience they are also linked in ReactQuill's `dist` folder.
Here's an example using [style-loader](https://www.npmjs.com/package/style-loader) for Webpack, or `create-react-app`, that will automatically inject the styles on the page:
```
import 'react-quill/dist/quill.snow.css';
```
The styles are also available via CDN:
```
<link
  rel="stylesheet"
  href="https://unpkg.com/react-quill@1.3.3/dist/quill.snow.css"
/>
```
### Custom Toolbar
#### Default Toolbar Elements
The [Quill Toolbar Module](http://quilljs.com/docs/modules/toolbar/) API provides an easy way to configure the default toolbar icons using an array of format names.
Example Code

```
class MyComponent extends Component {
  constructor(props) {
    super(props);
    this.state = {
      text: "",
    }
  }

  modules = {
    toolbar: [
      [{ 'header': [1, 2, false] }],
      ['bold', 'italic', 'underline','strike', 'blockquote'],
      [{'list': 'ordered'}, {'list': 'bullet'}, {'indent': '-1'}, {'indent': '+1'}],
      ['link', 'image'],
      ['clean']
    ],
  },

  formats = [
    'header',
    'bold', 'italic', 'underline', 'strike', 'blockquote',
    'list', 'bullet', 'indent',
    'link', 'image'
  ],

  render() {
    return (
      <div className="text-editor">
        <ReactQuill theme="snow"
                    modules={this.modules}
                    formats={this.formats}>
        </ReactQuill>
      </div>
    );
  }
}

export default MyComponent;
```