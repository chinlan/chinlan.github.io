---
layout: post
title: "Try react-autosuggest with redux-form"
description: ""
categories: [react]
tags: [react]
redirect_from:
  - /2020/06/23/
---

(Using Next.js, SSR)
```
npm install --save react-autosuggest
```

/components/common/InputAutoSuggestBlock.js
~~~react
import PropTypes from "prop-types";
import * as React from "react";
import Autosuggest from "react-autosuggest";
import Suggestion from "../../models/suggestion";
import { RegionTypes } from "../../constants/RegionTypes";

const propTypes = {
  label: PropTypes.shape({
    children: PropTypes.string.isRequired,
  }),
  input: PropTypes.shape({
    onChange: PropTypes.func.isRequired,
  }).isRequired,
  fetchRegionSuggest: PropTypes.func.isRequired,
  clearSuggestions: PropTypes.func.isRequired,
  suggestions: PropTypes.arrayOf(PropTypes.instanceOf(Suggestion)).isRequired,
  change: PropTypes.func.isRequired,
  onFocus: PropTypes.func,
  onSelected: PropTypes.func.isRequired,
  onClear: PropTypes.func,
  inputError: PropTypes.shape({
    meta: PropTypes.object.isRequired,
  }).isRequired,
  inputBlockRef: PropTypes.func.isRequired,
};

export default class InputAutoSuggestBlock extends React.Component {
  // react-autosuggestのonChangeを使うと日本語文字入力が途中でおかしくなる
  // nihon -> 日本
  // と入力したいが
  // nihon -> にhおn
  // とかになる(文字入力が確定され、2byte入力ができなくなる)
  onChangeRegion = (event) => {
    const { value } = event.target;
    this.props.input.onChange(event);
    this.onSuggestionsFetchRequested({ value, reason: "input-changed" });
    if (value === "" && this.props.onClear) {
      this.props.onClear();
    }
  };

  onSuggestionSelected = (event, data) => {
    this.props.input.onChange(data.suggestion.name);
    this.props.onSelected(data.suggestion);
  };

  getSuggestionValue = (suggestion) => {
    return suggestion.id;
  };

  renderSuggestion = (suggestion) => {
    let iconType = "place";
    switch (suggestion.regionType) {
      case RegionTypes.city:
        iconType = "location_city";
        break;
      case RegionTypes.prefecture:
        iconType = "location_on";
        break;
      default:
    }
    return (
      <div className="suggest">
        <i className="material-icons icon-suggest">{iconType}</i>
        {suggestion.name}
        {suggestion.label && (
          <span className="suggestion-label">{suggestion.label}</span>
        )}
      </div>
    );
  };

  // request example: {value: "t", reason: "input-changed"}
  onSuggestionsFetchRequested = (request) => {
    if (!request.value) {
      return;
    }
    if (
      request.reason !== "input-changed" &&
      request.reason !== "input-focused"
    ) {
      return;
    }
    this.props.fetchRegionSuggest(request.value);
  };

  onSuggestionsClearRequested = () => {
    this.props.clearSuggestions();
  };

  onClearInput = (event) => {
    event.preventDefault();
    if (this.inputRef) {
      this.inputRef.value = "";
    }
    if (this.props.onClear) {
      this.props.onClear();
    }
  };

  // react-autosuggestのonChangeを使うと日本語文字入力が途中でおかしくなる
  // react-autosuggestのonChangeを使わないように修正
  renderInputComponent = (inputProps) => {
    return (
      <>
        <input {...inputProps} onChange={this.onChangeRegion} />
        {inputProps.value ? (
          <button className="button-clear" onClick={this.onClearInput}>
            <i className="material-icons">close</i>
          </button>
        ) : (
          ""
        )}
      </>
    );
  };

  setInputRef = (node) => {
    if (node && node.input) {
      this.inputRef = node.input;
    }
  };

  render() {
    const { label } = this.props;
    const { input, inputError } = this.props;
    const error = !!(inputError.meta.touched && inputError.meta.error);

    const InputProps = {
      ...input,
      onFocus: this.props.onFocus,
      onChange: this.onChangeRegion,
      className: error ? "error" : "autosuggest-input",
    };
    const theme = {
      container: "input-container",
      suggestionsContainer: "suggestions-container",
      suggestionsContainerOpen: "suggestions-container-open",
      suggestionsList: "suggestions-list",
      suggestionHighlighted: "suggestion-highlighted",
    };

    return (
      <div className="region-autosuggest" ref={this.props.inputBlockRef}>
        {label && <label>{label.children}</label>}
        <Autosuggest
          suggestions={this.props.suggestions}
          renderInputComponent={this.renderInputComponent}
          onSuggestionsFetchRequested={this.onSuggestionsFetchRequested}
          onSuggestionsClearRequested={this.onSuggestionsClearRequested}
          getSuggestionValue={this.getSuggestionValue}
          inputProps={InputProps}
          theme={theme}
          renderSuggestion={this.renderSuggestion}
          onSuggestionSelected={this.onSuggestionSelected}
          highlightFirstSuggestion
          ref={this.setInputRef}
        />
      </div>
    );
  }
}

InputAutoSuggestBlock.propTypes = propTypes;

~~~

/component/common/RegionBlock.js
~~~ react
import PropTypes from "prop-types";
import * as React from "react";
import { Field } from "redux-form";
import { withTranslation } from "react-i18next";
// import { required } from "../../validators/required";
import InputAutoSuggestBlock from "./InputAutoSuggestBlock";
import Suggestion from "../../models/suggestion";
import Scroll, { defaultPadding } from "../../utils/Scroll";
import { FORM_SEARCH_JOB } from "../../constants/FormTypes";

const propTypes = {
  suggestions: PropTypes.arrayOf(PropTypes.instanceOf(Suggestion)).isRequired,
  t: PropTypes.func.isRequired,
  actions: PropTypes.shape({
    fetchRegionSuggest: PropTypes.func.isRequired,
    clearSuggestions: PropTypes.func.isRequired,
    change: PropTypes.func.isRequired,
  }),
};

const scrollDuration = 300;

class RegionBlock extends React.Component {
  inputBlock;

  setInputBlock = (node) => {
    this.inputBlock = node;
  };

  onFocusInput = () => {
    if (this.inputBlock) {
      Scroll.scrollToElementInsideParent(
        this.inputBlock,
        scrollDuration,
        defaultPadding
      );
    }
  };

  renderRegion = ({ input, meta }) => {
    const { t } = this.props;
    return (
      <InputAutoSuggestBlock
        label={{
          children: t("label:common.region"),
        }}
        input={{
          ...input,
          type: "text",
          placeholder: t("label:input.placeholderInput"),
        }}
        inputError={{
          meta,
        }}
        fetchRegionSuggest={this.props.actions.fetchRegionSuggest}
        clearSuggestions={this.props.actions.clearSuggestions}
        suggestions={this.props.suggestions}
        onSelected={this.onSelected}
        onFocus={this.onFocusInput}
        onClear={this.onClearInput}
        inputBlockRef={this.setInputBlock}
        change={this.props.actions.change}
      />
    );
  };

  onSelected = (suggestion) => {
    this.props.actions.change(FORM_SEARCH_JOB, "region_ids", suggestion.id);
  };

  onClearInput = () => {
    this.props.actions.change(FORM_SEARCH_JOB, "region", "");
    this.props.actions.change(FORM_SEARCH_JOB, "region_ids", null);
  };

  render() {
    return (
      <div className="input">
        <Field
          name="region"
          // validate={required}
          suggestions={this.props.suggestions}
          component={this.renderRegion}
        />
      </div>
    );
  }
}
RegionBlock.propTypes = propTypes;

export default withTranslation(["label"])(RegionBlock);
~~~

`react-autosuggest` will directly display the value of field/input in its input.
That is, even if we want to pass integer `id` rather than string `name` to server,
we can not write
~~~ html
<Field name="region_ids" ... />
~~~
or user will have to chose region with ids, which definitely is not a good experience.

Instead of using ids directly, we write
~~~ html
<Field name="region" ... />
~~~
and using `onSuggestionSelected` function which is provided by `react-autosuggest` to set the display content to string `name`.

~~~ react
onSuggestionSelected = (event, data) => {
  this.props.input.onChange(data.suggestion.name);
  this.props.onSelected(data.suggestion);
};
~~~

Then in `onSelected`, we use the `change` function which is provided by `redux-form` to manually update the value of `region_ids`.
~~~ react
onSelected = (suggestion) => {
  this.props.actions.change(FORM_SEARCH_JOB, "region_ids", suggestion.id);
};
~~~

Now we will have 2 contents within the form, `region` and `region_ids`, though server is only going to need `region_ids`.

Notice that `react-autosuggest` can only do with single select, when server-side needs an array format value, will need to adjust the value in `onSubmit` of the form.
~~~ react
onSubmit = (values, dispatch, props) => {
  // react-autosuggest can only deal with string value
  const adjustedValues = Object.assign({}, values);
  if (values.region_ids) {
    adjustedValues.region_ids = [values.region_ids];
  }
  // ...... do some more adjust and call API
};
~~~
The `InputProps` of `AutoSuggest` can only be a string, so do not adjust the values directly, assign it to another new variable.



