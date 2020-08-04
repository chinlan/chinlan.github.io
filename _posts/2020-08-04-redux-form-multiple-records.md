---
layout: post
title: "Redux-form: create and update using the same form component"
description: ""
categories: [devops]
tags: []
redirect_from:
  - /2020/08/04/
---
Goal: To create and update records on the same page and using the same form component.
      Have one `create` button above the record list, and an `update` button with every record in the list. When button is clicked, will display a form in a modal.

Forms will need specific name to be recognized connecting to redux-form store.
By passing `form` when rendering form component, not when exporting the form component, we can assign specific name to each form.

/components/CreateRecordButton.js
~~~js
import PropTypes from "prop-types";
import * as React from "react";
import { withTranslation } from "react-i18next";
import Alert from "react-s-alert";
import ErrorMessage from "../common/ErrorMessage";
import { ErrorOption, SuccessOption } from "../../constants/AlertOption";
import ParentRecord from "../../models/parentRecord";
import RecordFormModalDialog from "./modal/RecordFormModalDialog";

const propTypes = {
  record: PropTypes.instanceOf(ParentRecord),
  t: PropTypes.func,
  actions: PropTypes.shape({
    postRecord: PropTypes.func.isRequired,
  }),
};

class CreateRecordButton extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      displayedModal: false,
    };
  }

  onClick = () => {
    this.setState({
      displayedModal: true,
    });
  };

  onClickCancel = () => {
    this.setState({
      displayedModal: false,
    });
  };

  onClickSave = (param) => {
    if (!this.props.parentRecord) return {};
    const submitParam = {
      value: param.value,
      type: param.type,
      parent_record_id: this.props.parentRecord.id,
    };
    return this.props.actions.postRecord(submitParam).then((res) => {
      if (res.error) {
        Alert.error(<ErrorMessage message={res.errors} />, ErrorOption);
      } else {
        this.setState({
          displayedModal: false,
        });
        Alert.success("Record created successfully.", SuccessOption);
      }
    });
  };

  render() {
    const { t } = this.props;
    return (
      <div>
        <div className="attribute-selectbox">
          <button
            className="button button--accent button-record"
            onClick={this.onClick}
          >
            <span>{t("button:createRecord")}</span>
          </button>
        </div>
        <RecordFormModalDialog
          saveHandler={this.onClickSave}
          cancelHandler={this.onClickCancel}
          displayed={this.state.displayedModal}
          form={'formRecord'}
          initialValues={{
            type: null,
            value: null,
          }}
        />
      </div>
    );
  }
}

CreateRecordButton.propTypes = propTypes;
export default withTranslation(["label", "model"])(CreateRecordButton);

~~~

components/UpdateRecordButton.js
~~~js
import PropTypes from "prop-types";
import * as React from "react";
import { withTranslation } from "react-i18next";
import Alert from "react-s-alert";
import ErrorMessage from "../common/ErrorMessage";
import { ErrorOption, SuccessOption } from "../../constants/AlertOption";
import Record from "../../models/record";
import RecordFormModalDialog from "./modal/RecordFormModalDialog";

const propTypes = {
  record: PropTypes.instanceOf(Record).isRequired,
  t: PropTypes.func,
  actions: PropTypes.shape({
    putRecord: PropTypes.func.isRequired,
  }),
};

class UpdateRecordButton extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      displayedModal: false,
    };
  }

  onClick = () => {
    this.setState({
      displayedModal: true,
    });
  };

  onClickCancel = () => {
    this.setState({
      displayedModal: false,
    });
  };

  onClickSave = (param) => {
    if (!this.props.record) return {};
    const recordId = this.props.record.id;
    const submitParam = {
      value: param.value,
      type: param.type,
    };
    return this.props.actions.putRecord(recordId, submitParam).then((res) => {
      if (res.error) {
        Alert.error(<ErrorMessage message={res.errors} />, ErrorOption);
      } else {
        this.setState({
          displayedModal: false,
        });
        Alert.success("Record updated successfully.", SuccessOption);
      }
    });
  };

  render() {
    const { t, record } = this.props;
    return (
      <div>
        <button
          className="button button--accent button-recordUpdate"
          onClick={this.onClick}
        >
          <span>{t("button:update")}</span>
        </button>
        <RecordFormModalDialog
          saveHandler={this.onClickSave}
          cancelHandler={this.onClickCancel}
          displayed={this.state.displayedModal}
          form={`record[${record.id}]`}
          initialValues={{
            process_type: record.processType,
            memo: record.memo,
          }}
        />
      </div>
    );
  }
}

UpdateRecordButton.propTypes = propTypes;
export default withTranslation(["label", "model"])(UpdateRecordButton);

~~~

When exporting the form component, add `enableReinitialize: true` setting.
And `this.props.reset();` when `onSubmit`.

modal/RecordFormModalDialog.js
~~~js
import PropTypes from "prop-types";
import * as React from "react";
import { withTranslation } from "react-i18next";
import { Field, reduxForm } from "redux-form";
import ModalDialogWithClose from "../../common/modal/ModalDialogWithClose";
import ModalTitle from "../../common/modal/ModalTitle";
import { required } from "../../../validators/required";
import InputError from "../../common/InputError";
import RecordTypeBlock from "../../common/RecordTypeBlock";

const propTypes = {
  saveHandler: PropTypes.func.isRequired,
  cancelHandler: PropTypes.func.isRequired,
  displayed: PropTypes.bool.isRequired,
  change: PropTypes.func.isRequired,
  handleSubmit: PropTypes.func.isRequired,
  submitting: PropTypes.bool,
  t: PropTypes.func,
  initialValues: PropTypes.shape({
    value: PropTypes.string,
    type: PropTypes.string,
  }).isRequired,
};

const modalText = {
  title: "component:chat.recordFormTitle",
  description: "component:chat.recordFormDescription",
  save: "button:save",
  cancel: "button:cancel",
};

class RecordFormModalDialog extends React.Component {
  onSubmit = (param) => {
    this.props.reset();
    return this.props.saveHandler(param);
  };

  renderValue = ({ input, meta }) => {
    return (
      <div className="">
        <label>Value</label>
        <textarea className="textarea" rows="6" {...input} />
        <InputError meta={meta} />
      </div>
    );
  };

  render() {
    const {
      cancelHandler,
      displayed,
      change,
      handleSubmit,
      submitting,
      t,
    } = this.props;

    return (
      <ModalDialogWithClose
        displayed={displayed}
        closeHandler={cancelHandler}
        className="modal-record"
      >
        <form onSubmit={handleSubmit(this.onSubmit)}>
          <div>
            <ModalTitle>{t(modalText.title)}</ModalTitle>
            <div className="modal-text">{t(modalText.description)}</div>
            <div className="modal-record__input-block">
              <RecordTypeBlock />
              <div className="record-value">
                <Field
                  name="value"
                  component={this.renderValue}
                  placeholder={t(`label:input.placeholderHope`)}
                />
              </div>
            </div>
          </div>
          <div className="modal-dialog__button-container">
            <button
              className="button button--alert modal-button"
              type="submit"
              disabled={submitting}
            >
              {t(modalText.save)}
            </button>
            <button
              className="button modal-button"
              onClick={cancelHandler}
              type="button"
            >
              {t(modalText.cancel)}
            </button>
          </div>
        </form>
      </ModalDialogWithClose>
    );
  }
}

RecordFormModalDialog.propTypes = propTypes;
export default reduxForm({
  enableReinitialize: true,
})(withTranslation(["component", "button"])(RecordFormModalDialog));

~~~

[Reference](https://stackoverflow.com/questions/37456526/how-to-embed-the-same-redux-form-multiple-times-on-a-page/37464048#37464048)

