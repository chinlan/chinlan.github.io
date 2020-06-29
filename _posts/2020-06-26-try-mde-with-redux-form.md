---
layout: post
title: "Try mde with redux-form"
description: ""
categories: [react]
tags: [react]
redirect_from:
  - /2020/06/26/
---

(Using Next.js, SSR)

```
npm install --save react-mde
```

/components/common/MDEditorBlock.js
~~~react
import React, { useState } from "react";
import PropTypes from "prop-types";
import ReactMde from "react-mde";
import * as Showdown from "showdown";

const propTypes = {
  inputClassName: PropTypes.string,
  input: PropTypes.shape({}).isRequired,
};

const converter = new Showdown.Converter({
  tables: true,
  simplifiedAutoLink: true,
  strikethrough: true,
  tasklists: true,
});

const MDEditorBlock = (props) => {
  const [selectedTab, setSelectedTab] = useState("write");
  const onChangeValue = (value) => {
    props.input.onChange(value);
  };

  return (
    <div className="container">
      <ReactMde
        value={props.input.value}
        onChange={onChangeValue}
        selectedTab={selectedTab}
        onTabChange={setSelectedTab}
        generateMarkdownPreview={(markdown) =>
          Promise.resolve(converter.makeHtml(markdown))
        }
        classes={{
          reactMde: props.inputClassName,
        }}
      />
    </div>
  );
};

MDEditorBlock.propTypes = propTypes;

export default MDEditorBlock;
~~~


