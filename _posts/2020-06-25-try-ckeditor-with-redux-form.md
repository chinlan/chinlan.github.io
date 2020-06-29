---
layout: post
title: "Try ckeditor with redux-form"
description: ""
categories: [react]
tags: [react]
redirect_from:
  - /2020/06/25/
---

(Using Next.js, SSR)
```
npm install --save @ckeditor/ckeditor5-react
npm install --save @ckeditor/ckeditor5-build-classic
```

/components/common/CKEditorBlock.js
~~~react
import React from "react";
import PropTypes from "prop-types";

const propTypes = {
  inputClassName: PropTypes.string,
  input: PropTypes.shape({}).isRequired,
};

class CKEditorBlock extends React.Component {
  render() {
    const { inputClassName, input } = this.props;

    // 'window is not defined error' occurs due to SSR with next.js,
    // have to require CKEditor in the render function.
    if (typeof window === "undefined") {
      return <p>isLoading</p>;
    } else {
      const CKEditor = require("@ckeditor/ckeditor5-react");
      const ClassicEditor = require("@ckeditor/ckeditor5-build-classic");
      return (
        <CKEditor
          editor={ClassicEditor}
          data={input.value}
          onInit={(editor) => {
            // can store the "editor" and use when it is needed.
            console.log("Editor is ready to use!", editor);
          }}
          onChange={(event, editor) => {
            const data = editor.getData();
            console.log({ event, editor, data });
            return input.onChange(editor.getData());
          }}
          className={inputClassName}
        />
      );
    }
  }
}

CKEditorBlock.propTypes = propTypes;

export default CKEditorBlock;
~~~

next.jsでSSRしてる時、CKEditorにwindow is not definedエラーが起こります、
なのでファイルの最初からimportすることができなくて、renderの中（windowがもう存在する）にrequireすることにしました。
(eslintに怒られて、一旦@typescript-eslint/no-var-requiresをoffにします）

