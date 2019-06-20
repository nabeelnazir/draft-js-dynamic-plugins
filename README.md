# draft-js-plugins-dynamic
This will explain how to use multiple draft-js-plugins dynamically in one page in react js. 
```
import React, { Component } from 'react';
import withStyles from 'isomorphic-style-loader/lib/withStyles';
import Editor from 'draft-js-plugins-editor';
import createLinkPlugin from 'draft-js-anchor-plugin';
import createInlineToolbarPlugin from 'draft-js-inline-toolbar-plugin';
import draftDefaultStyles from './DraftDefault.css';
import editorStyles from './DraftEditor.css';
import buttonStyles from './buttonStyles.css';
import toolbarStyles from './toolbarStyles.css';
import linkStyles from './linkStyles.css';

import ss from 'draft-js-inline-toolbar-plugin/lib/plugin.css';
import { convertToRaw, EditorState } from 'draft-js';
import { stateToHTML } from 'draft-js-export-html';
import { stateFromHTML } from 'draft-js-import-html';
import { UnorderedListButton, OrderedListButton } from 'draft-js-buttons';

import { connect } from 'react-redux';
import { compose } from 'redux';
import { withCookies } from 'react-cookie';

const linkPlugin1 = createLinkPlugin({
  theme: linkStyles,
  placeholder: 'Enter a valid URL and press enter.',
  linkTarget: '_blank',
});

class DraftEditorSingle extends Component {
  constructor(props) {
    super(props);
    this.inlineTool = createInlineToolbarPlugin({
      theme: { buttonStyles, toolbarStyles },
    });
    this.linkPlugin = createLinkPlugin({
      theme: linkStyles,
      placeholder: 'Enter a valid URL and press enter.',
      linkTarget: '_blank',
    });
    const contentState = stateFromHTML(props.description);
    this.state = {
      editorState: EditorState.createWithContent(contentState),
    };
    this.onChange = this.onChange.bind(this);
  }

  onChange(editorState) {
    this.setState({
      editorState,
    });
    const content = editorState.getCurrentContent();
    const dataToSaveBackend = convertToRaw(content);
    let options = {
      entityStyleFn: entity => {
        const entityType = entity.get('type').toLowerCase();
        if (entityType === 'link') {
          const data = entity.getData();
          return {
            element: 'a',
            attributes: {
              href: data.url,
              target: '_blank',
            },
            style: {
              // Put styles here...
            },
          };
        }
      },
    };
    this.props.descriptionEditChild({
      target: { value: stateToHTML(content, options) },
    });
  }

  focus = () => {
    this.editor.focus();
  };


  render() {
    const plugins = [this.inlineTool, this.linkPlugin];
    const { InlineToolbar } = this.inlineTool;
    const { LinkButton } = this.linkPlugin;
    return (
      <div className={"MyEditor"} onClick={this.focus}>
        <div>
          <Editor
            editorState={this.state.editorState}
            onChange={this.onChange}
            plugins={plugins}
            ref={element => {
              this.editor = element;
            }}
            placeholder={"Enter data..."}
          />
          <InlineToolbar>
            {externalProps => (
              <div>
                <UnorderedListButton {...externalProps} />
                <OrderedListButton {...externalProps} />
                <LinkButton {...externalProps} />
              </div>
            )}
          </InlineToolbar>
        </div>
      </div>
    );
  }
}

const mapStateToProps = state => ({});

export default compose(
  withCookies,
  withStyles(
    draftDefaultStyles,
    ss,
    editorStyles,
    buttonStyles,
    toolbarStyles,
    linkStyles,
  ),
  connect(
    mapStateToProps,
    null,
  ),
)(DraftEditorSingle);

```

In Above React JS component we used https://github.com/draft-js-plugins/draft-js-plugins, draft-js-plugins is a wrapper on draf-js plugin. 

The above component will help in building Inline toolbar for your editor with Ordered bulltets, Unordered numbers and a Link button, you can also add more plugins. Moreover, you can use this component for your form builder or a form which has dynamic fields i.e. you can create dynamic multiple editors within one page by calling the component like this:

```
<DraftEditorSingle
  description={this.props.description}
  descriptionEditChild={this.props.descriptionEdit}
/>

```

In very above snippet, `description` is actually the value (content) of your editor and `descriptionEditChild` is a function which will be responsible for storing new value of your content. Simple!

Please feel free to ask or add anything to this repository :)
