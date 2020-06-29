---
layout: post
title: "Try uploading multiple files using react-dropzone with redux-form"
description: ""
categories: [react]
tags: [react]
redirect_from:
  - /2020/06/27/
---

(Using Next.js, SSR)

```
npm install --save react-dropzone
```

~~~react
import Dropzone from "react-dropzone";

const propTypes = {
  ...reduxFormPropTypes,
};

class MyComponent extends React.Component {
  onSubmit = (values) => {
    // adjust the values if needed, call API, etc...
  };

  dropzoneFileInput = (field) => {
    const [uploadedFiles, setUploadedFiles] = useState([]);

    const readUploadedFileAsDataURL = (acceptedFile) => {
      const reader = new FileReader();
      return new Promise((resolve, reject) => {
        reader.onerror = () => {
          reader.abort();
          reject(new DOMException("Problem parsing input file."));
        };
        reader.onload = () => {
          resolve(reader.result);
        };
        reader.readAsDataURL(acceptedFile);
      });
    };

    const readFiles = (acceptedFiles) => {
      const objects = [];
      acceptedFiles.map(async (acceptedFile) => {
        const fileDataURL = await readUploadedFileAsDataURL(acceptedFile);
        objects.push({ name: acceptedFile.name, file: fileDataURL });
      });
      setUploadedFiles([...acceptedFiles]);
      field.input.onChange(objects);
    };

    const removeFile = (file) => () => {
      const newFiles = [...uploadedFiles];
      newFiles.splice(newFiles.indexOf(file), 1);
      readFiles(newFiles);
    };

    const thumbnails = uploadedFiles.map((file) => (
      <li key={file.path}>
        {file.path} - {file.size} bytes{" "}
        {
          //<img src={URL.createObjectURL(file)} />
        }
        <button onClick={removeFile(file)}>x</button>
      </li>
    ));

    return (
      <div className="chat-message-inputblock-email__image" id="dropzone">
        <Dropzone name={field.name} onDrop={readFiles} multiple>
          {({ getRootProps, getInputProps }) => (
            <div {...getRootProps()}>
              {" "}
              <input {...getInputProps()} />
              ドラッグアンドドロップか、クリックしてファイルを選択してください
            </div>
          )}
        </Dropzone>
        {field.meta.touched && field.meta.error && (
          <span className="error">{field.meta.error}</span>
        )}
        <ul className="chat-message-inputblock-email__image" id="preview">
          {thumbnails}
        </ul>
      </div>
    );
  };

  render() {
    return (
      <div>
        <Form
          onSubmit={this.onSubmit}
          className="chat-message-inputblock__form"
          disabled={!selectedChatUnit}
        >
          // ... other fields ...
          <div className="chat-message-inputblock-email__container">
            <div className="chat-message-inputblock-email__item_file">
              {t(`label:input.attachments`)}
            </div>
            <Field
              name="preview.email.attachments"
              component={this.dropzoneFileInput}
              type="file"
            />
          </div>
        </form>
      </div>
    );
  }
}
~~~

If using json format to pass the request params, will need to encoding the attachment content to a dataUrl using FileReader.
Because FileReader needs time to process the result, will need to add a Promise to get the encoded dataUrl result in `reader.onload()` block.

The `setState` part needs to be revised to using global store (add a container etc.) later.
Since `setUploadedFiles` cannot be called in `onSubmit()` function, the thumbnail preview of the uploaded attachments cannot be removed after submitting the form.

