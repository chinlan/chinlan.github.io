---
layout: post
title: "Try react-select with redux-form"
description: ""
categories: [react]
tags: [react]
redirect_from:
  - /2020/06/24/
---

(Using Next.js, SSR)
```
npm install --save react-select
```

/components/common/ReactSelectMultipleBlock.js
~~~ react
import PropTypes from "prop-types";
import * as React from "react";
import { withTranslation } from "react-i18next";
import Select from "react-select";
import { FORM_SEARCH_JOB } from "../../constants/FormTypes";

const propTypes = {
  input: PropTypes.any.isRequired,
  styles: PropTypes.string,
  label: PropTypes.shape({
    name: PropTypes.string,
  }),
  options: PropTypes.array.isRequired,
  t: PropTypes.func.isRequired,
  attrName: PropTypes.string.isRequired,
  actions: PropTypes.shape({
    change: PropTypes.func.isRequired,
  }),
};

class ReactSelectMultipleBlock extends React.Component {
  render() {
    const { input, options, t, attrName, label } = this.props;
    const { styles = "" } = this.props;

    const onChangeValue = (value) => {
      input.onChange(value);
      const adjustedValue = value === null ? value : value.map((v) => v.value);
      this.props.actions.change(FORM_SEARCH_JOB, attrName, adjustedValue);
    };

    return (
      <div className={`chat-search_plan_select ${styles}`}>
        {label && label.name && (
          <label htmlFor={input.name}>{label.name}</label>
        )}
        <Select
          {...input}
          onChange={onChangeValue}
          onBlur={() => {
            input.onBlur([...input.value]);
          }}
          options={options}
          isMulti
          placeholder={t("component:common.selectPlaceholder")}
        />
      </div>
    );
  }
}

ReactSelectMultipleBlock.propTypes = propTypes;

export default withTranslation(["component"])(ReactSelectMultipleBlock);
~~~

/components/common/JobCategoryBlock.js
~~~ react
import React from "react";
import PropTypes from "prop-types";
import { withTranslation } from "react-i18next";
import { Field } from "redux-form";
import ReactSelectMultipleBlock from "../../containers/common/ReactSelectMultipleBlock";

const propTypes = {
  t: PropTypes.func.isRequired,
  jobCategories: PropTypes.any.isRequired,
};

class JobCategoryBlock extends React.Component {
  render() {
    const { t, jobCategories } = this.props;
    const options = [];
    jobCategories.forEach((category) => {
      options.push({ label: category.name, value: category.id });
    });

    return (
      <Field
        name="job_categories"
        component={ReactSelectMultipleBlock}
        options={options}
        styles="input input-job-category width-full"
        label={{
          name: t("label:job.jobCategory"),
        }}
        attrName="job_category_ids"
      />
    );
  }
}

JobCategoryBlock.propTypes = propTypes;

export default withTranslation(["constant", "label"])(JobCategoryBlock);
~~~

`react-select` will return value in an object form `{value: xxx, label: xxx}`,
to get the value server needs as an array of just values,
we do the same thing we did when dealing with `react-autosuggest`:
Srtting a different name for the field, and using `redux-form` `change` function to manually update the value of `xxx_ids` in the formValues.

[reference](http://chinlan.github.io/blog/2020/06/23/try-react-autosuggest-with-redux-form/)
