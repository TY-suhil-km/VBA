<template>
  <div
    class="resp-textbox"
    v-on="eventStoppers()"
    :title="properties.ControlTipText"
    :tabindex="properties.TabIndex"
    :style="cssStyleProperty"
    @click="textBoxClick"
    @mousedown="controlEditMode"
    @keydown.enter.self="setContentEditable($event, true)"
    @mouseover="updateMouseCursor"
    @focus="closeTextMenu"
  >
    <div
      data-gramm="false"
      :style="divCssStyleProperty"
      @dblclick="dblclick($event)"
      :readonly="properties.Locked"
      class="text-box-design"
    >
      <span
        id="initialRef"
        ref="editableTextBoxRef"
        :tabindex="properties.TabIndex"
        :style="divCssStyleProperty"
        :contenteditable="!getDisableValue"
        v-cursorDirective="{
        start: data.properties.CursorStartPosition,
        end: data.properties.CursorEndPosition,
        pwdCharType: properties.PasswordChar,
      }"
        @dragstart="dragBehavior"
        @keyup="properties.PasswordChar !== '' ? handleDelete($event) : null"
        @keydown.tab="tabKeyBehavior"
        @input="updateValueProp($event)"
        @keydown.enter="enterKeyBehavior"
        @keydown.escape.exact="releaseEditMode"
        @contextmenu="isEditMode ? setCursorStartEndPosition($event) : parentConextMenu($event)"
        @focus="closeTextMenu"
        @blur="handleBlur($event)"
      ></span>
    </div>
  </div>
</template>

<script lang="ts">
import {
  Component,
  Vue,
  Prop,
  Ref,
  Watch,
  Emit,
  Mixins
} from 'vue-property-decorator'
import FdControlVue from '@/api/abstract/FormDesigner/FdControlVue'
import { DirectiveBinding } from 'vue/types/options'
import { EventBus } from '@/FormDesigner/event-bus'
import FDEditableText from '@/FormDesigner/components/atoms/FDEditableText/index.vue'

@Component({
  name: 'FDTextBox',
  components: {
    FDEditableText
  },
  filters: {
    /**
     * @description filter to show passwordChar instead of original text
     * @function passwordFilter
     * @param value text to be filtered
     * @param password passwordChar value to filter the text
     * @param text  TextBox properties Text Value
     */
    passwordFilter (value: string, password: string, text: string) {
      if (password !== '' && text !== '') {
        let filteredValue: string = ''
        for (let index = 0; index < text.length; index++) {
          filteredValue = filteredValue + password[0]
        }
        return filteredValue
      } else {
        return value
      }
    }
  },
  directives: {
    cursorDirective: {
      /**
       * @description  called after the containing component’s VNode has updated
       * updates selection start and end cursor positon when data model value changes
       * to maintain cursor position
       * @function update Hook Functions i.e, (bind, inserted, update, componentUpdated, unbind)
       * @param event The element the directive is bound to. This can be used to directly manipulate the DOM.
       * @param vnode The virtual node produced by Vue’s compiler
       */
      update (event: HTMLElement, vnode: DirectiveBinding) {
        if (vnode.value.pwdCharType !== '') {
          (event as HTMLFormElement).selectionStart = vnode.value.start;
          (event as HTMLFormElement).selectionEnd = vnode.value.end
        } else {
          return undefined
        }
      }
    }
  }
})
export default class FDTextBox extends Mixins(FdControlVue) {
  @Ref('hideSelectionDiv') readonly hideSelectionDiv!: HTMLDivElement;
  @Ref('autoSizeTextarea') readonly autoSizeTextarea!: HTMLLabelElement;
  @Ref('textareaRef') readonly textareaRef!: HTMLDivElement;
  @Ref('editableTextBoxRef') editableTextBoxRef!: HTMLSpanElement;

  $el: HTMLDivElement;
  originalText: string = '';
  trimmedText: string = '';
  prevSelection: number = 0;
  isUpdatingText: boolean = false;
  dblclick (e: Event) {
    //   let newSelectionStart = 0
    //   const eTarget = e.target as HTMLDivElement
    //   for (let i = eTarget.selectionStart; i > 0; i--) {
    //     if (eTarget.value[i - 1] === ' ' || eTarget.value[i - 1] === undefined) {
    //       newSelectionStart = i
    //       break
    //     }
    //   }
    //   this.textareaRef.setSelectionRange(newSelectionStart, eTarget.selectionEnd)
  }
  get getDisableValue () {
    if (this.isRunMode) {
      return this.properties.Enabled === false || this.properties.Locked
    } else if (this.isEditMode) {
      if (this.properties.Locked) {
        return true
      } else {
        return false
      }
    } else {
      return true
    }
  }
  // @Watch('properties.HideSelection')
  // updateIsTextBoxDisabledPropWhenHideSelection () {
  //   if (this.properties.HideSelection) {
  //     this.textareaRef.style.display = 'block'
  //     this.hideSelectionDiv.style.display = 'none'
  //   } else {
  //     this.textareaRef.style.display = 'none'
  //     this.hideSelectionDiv.style.display = 'block'
  //   }
  // }
  get fontCssStyleProperty () {
    const controlProp = this.properties
    const font: font = controlProp.Font
      ? controlProp.Font
      : {
        FontName: 'Arial',
        FontSize: 10,
        FontItalic: true,
        FontBold: true,
        FontUnderline: true,
        FontStrikethrough: true,
        FontStyle: 'Arial'
      }
    return {
      fontFamily: font.FontStyle! !== '' ? this.setFontStyle : font.FontName!,
      fontSize: `${font.FontSize}px`,
      fontStyle: font.FontItalic || this.isItalic ? 'italic' : '',
      textDecoration:
        font.FontStrikethrough === true && font.FontUnderline === true
          ? 'underline line-through'
          : font.FontUnderline
            ? 'underline'
            : font.FontStrikethrough
              ? 'line-through'
              : '',
      textDecorationSkipInk: 'none',
      fontWeight: font.FontBold
        ? 'bold'
        : font.FontStyle !== ''
          ? this.tempWeight
          : '',
      fontStretch: font.FontStyle !== '' ? this.tempStretch : ''
    }
  }
  get divCssStyleProperty () {
    const controlProp = this.properties
    return {
      ...this.fontCssStyleProperty,
      whiteSpace: controlProp.WordWrap && controlProp.MultiLine ? 'break-spaces' : 'pre',
      wordBreak: controlProp.WordWrap && controlProp.MultiLine ? 'break-word' : 'normal',
      width: 'fit-content',
      height: 'fit-content',
      display: 'inline-block',
      padding: '1px',
      minWidth: '1px'
    }
  }
  /**
   * @description style object is passed to :style attribute in Textarea tag
   * dynamically changing the styles of the component based on properties
   * @function cssStyleProperty
   *
   */
  get cssStyleProperty () {
    const controlProp = this.properties
    let display = ''
    if (this.isRunMode) {
      display = controlProp.Visible
        ? controlProp.Width === 0 || controlProp.Height === 0
          ? 'none'
          : 'inline-block'
        : 'none'
    } else {
      display =
        controlProp.Width === 0 || controlProp.Height === 0
          ? 'none'
          : 'inline-block'
    }
    return {
      ...this.fontCssStyleProperty,
      left: `${controlProp.Left}px`,
      width: `${controlProp.Width}px`,
      height: `${controlProp.Height}px`,
      top: `${controlProp.Top}px`,
      borderColor: controlProp.BorderStyle === 1 ? controlProp.BorderColor : '',
      textAlign:
        controlProp.TextAlign === 0
          ? 'left'
          : controlProp.TextAlign === 1
            ? 'center'
            : 'right',
      backgroundColor: controlProp.BackStyle
        ? controlProp.BackColor
        : 'transparent',
      borderLeft:
        controlProp.BorderStyle === 1
          ? '1px solid ' + controlProp.BorderColor
          : controlProp.SpecialEffect === 2
            ? '2px solid gray'
            : controlProp.SpecialEffect === 3
              ? '1.5px solid gray'
              : controlProp.SpecialEffect === 4
                ? '0.5px solid gray'
                : '',
      borderRight:
        controlProp.BorderStyle === 1
          ? '1px solid ' + controlProp.BorderColor
          : controlProp.SpecialEffect === 1
            ? '2px solid gray'
            : controlProp.SpecialEffect === 4
              ? '1.5px solid gray'
              : controlProp.SpecialEffect === 3
                ? '0.5px solid gray'
                : '',
      borderTop:
        controlProp.BorderStyle === 1
          ? '1px solid ' + controlProp.BorderColor
          : controlProp.SpecialEffect === 2
            ? '2px solid gray'
            : controlProp.SpecialEffect === 3
              ? '1.5px solid gray'
              : controlProp.SpecialEffect === 4
                ? '0.5px solid gray'
                : '',
      borderBottom:
        controlProp.BorderStyle === 1
          ? '1px solid ' + controlProp.BorderColor
          : controlProp.SpecialEffect === 1
            ? '2px solid gray'
            : controlProp.SpecialEffect === 4
              ? '1.5px solid gray'
              : controlProp.SpecialEffect === 3
                ? '0.5px solid gray'
                : '',
      whiteSpace:
        controlProp.WordWrap && controlProp.MultiLine ? 'break-spaces' : 'pre',
      wordBreak:
        controlProp.WordWrap && controlProp.MultiLine ? 'break-word' : 'normal',
      color:
        controlProp.Enabled === true ? controlProp.ForeColor : this.getEnabled,
      cursor: this.controlCursor,
      display: display,
      overflowX: this.properties.AutoSize ? 'hidden' : this.getScrollBarX,
      overflowY: this.properties.AutoSize ? 'hidden' : this.getScrollBarY
    }
  }

  /**
   * @description updates the dataModel textBox object properties when user insert/delete text
   * inside textBox when passwordChar is set, updates text and values properties of textBox with entered character
   * @function handlePasswordChar
   * @param event its of type TextEvent
   * @event input
   *
   */
  handlePasswordChar (event: InputEvent | KeyboardEvent) {
    debugger
    const controlProp = this.properties
    const position = this.getCursorPos()
    let newData
    console.log(position)
    let text = this.properties.Text!
    let selectionDiff =
      (controlProp.CursorStartPosition as number) !==
      (controlProp.CursorEndPosition as number)
    if (!controlProp.MultiLine) {
      this.editableTextBoxRef.innerText = this.editableTextBoxRef.innerText.replace(
        /(\r\n|\n|\r)/gm,
        ''
      )
    }
    if (event.target instanceof HTMLSpanElement) {
      if (selectionDiff) {
        // selection
        newData =
        text.substring(0, controlProp.CursorStartPosition as number) +
        text.substring(controlProp.CursorEndPosition as number)
        if (!newData) {
          newData = event.target.innerText
        }
        this.updateDataModel({ propertyName: 'Text', value: newData })
        this.updateDataModel({ propertyName: 'Value', value: newData })
      } else if (text.length < event.target.innerText.length) {
        // insertion
        newData = [
          text.slice(0, position.startPosition - 1),
          event instanceof InputEvent ? event.data : '\t',
          text.slice(position.startPosition - 1)
        ].join('')
        this.updateDataModel({ propertyName: 'Text', value: newData })
        this.updateDataModel({ propertyName: 'Value', value: newData })
      } else if (text.length > event.target.innerText.length) {
        // deletion
        newData = [
          text.slice(0, position.startPosition),
          text.slice(position.startPosition + 1)
        ].join('')
        this.updateDataModel({ propertyName: 'Text', value: newData })
        this.updateDataModel({ propertyName: 'Value', value: newData })
      }
      this.updateDataModel({
        propertyName: 'CursorStartPosition',
        value: position.startPosition
      })
      this.updateDataModel({
        propertyName: 'CursorEndPosition',
        value: position.endPosition
      })
      if (controlProp.AutoTab && controlProp.MaxLength! > 0) {
        if (event.target instanceof HTMLDivElement) {
          if (event.target.innerText.length === controlProp.MaxLength) {
            EventBus.$emit('focusNextControlOnAutoTab')
          }
        }
      }
      if (controlProp.PasswordChar && (event instanceof InputEvent ? event.data : event.key === 'Tab')) {
        this.editableTextBoxRef.innerText = this.passwordCharFilter()
      }
    } else {
      throw new Error(
        'Expected HTMLTextAreaElement but found different element'
      )
    }
    // if (controlProp.PasswordChar !== '' && controlProp.Text !== '') {
    //   console.log(position.endPosition)
    //   debugger
    //   // this.setControlCaretPosition(true)
    //   Vue.nextTick(() => {
    //     (this.editableTextBoxRef as HTMLSpanElement).focus()
    //     const el = this.editableTextBoxRef
    //     const range = document.createRange()
    //     const sel = window.getSelection()!
    //     range.setStart(
    //       el.childNodes[0],
    //       position.endPosition + 1
    //     )
    //     range.collapse(true)
    //     sel.removeAllRanges()
    //     sel.addRange(range)
    //   })
    // }
  }
  /**
   * @description When user enters ctrl + enter cursor moves to next line
   * @function handleCtrlEnter
   * @param el its of type HTMLTextAreaElement
   * @param text new line character
   * @event keydown.enter.ctrl
   */
  handleCtrlEnter () {
    const { startPosition, endPosition } = this.getCursorPos()
    const selection = window.getSelection()!
    const range = selection.getRangeAt(0)
    const br = document.createTextNode('\n')
    range.deleteContents()
    range.insertNode(br)
    range.setStartAfter(br)
    range.setEndAfter(br)
    range.collapse(false)
    selection.removeAllRanges()
    selection.addRange(range)
  }
  /**
   * @description EnterKeyBehavior - if true when enter is pressed while editing the cursor moves to next line
   *  if false the cursor remains in same place
   * @function enterKeyBehavior
   * @param event its of type KeyboardEvent
   * @event keydown.enter
   */
  enterKeyBehavior (event: KeyboardEvent): boolean {
    if (this.properties.MultiLine) {
      if (event.ctrlKey) {
        // this.handleCtrlEnter()
        // const eTarget = event.target as HTMLTextAreaElement
        // this.updateDataModel({ propertyName: 'Value', value: eTarget.value })
        return true
      } else if (
        this.properties.EnterKeyBehavior &&
        this.properties.MultiLine
      ) {
        return true
      } else if (event.shiftKey) {
        return true
      }
    }
    event.preventDefault()
    return false
  }
  insertTab (event: KeyboardEvent) {
    debugger
    if (!window.getSelection) return
    // let range = document.createRange()
    // var sel = document.getSelection()!
    const { startPosition, endPosition } = this.getCursorPos()
    if (event.target instanceof HTMLSpanElement) {
      let val = event.target.innerText
      const selStart = startPosition
      var text = '\u0009'
      event.target.innerText = val.slice(0, selStart) + text + val.slice(endPosition)
      // console.log(event.target.textContent!.includes('\t'))
      // console.log(startPosition + 1, endPosition + 1, val.slice(endPosition))
      // console.log(event.target.innerText)
      // range.setStart(this.editableTextBoxRef.childNodes[0], startPosition + 1)
      // range.setEnd(this.editableTextBoxRef.childNodes[0], endPosition - 1)
      // range.collapse(true)
      // sel.removeAllRanges()
      // sel.addRange(range)
      const targetValue = event.target.innerText
      // this.handleDelete(event)
      this.isUpdatingText = true
      this.properties.PasswordChar === ''
        ? this.textAndValueUpdate(targetValue)
        : this.handlePasswordChar(event)
      // this.setControlCaretPosition(true)
    }
  }
  /**
   * @description  specifies how the control responds to the TAB key
   * when  TabKeyBehavior true in textBox tab spaces are added on press of tab Key
   * when TabKeyBehavior false in textBox pressing tab moves focus to next controls
   * @function tabKeyBehavior
   * @param event its of type MouseEvent
   * @event keydown.tab
   */
  tabKeyBehavior (event: KeyboardEvent) {
    console.log(this.getCursorPos().endPosition)
    debugger
    if (this.properties.TabKeyBehavior) {
      this.insertTab(event)
      event.preventDefault()
      if (this.properties.AutoSize) {
        this.updateAutoSize()
      }
    } else {
      EventBus.$emit('focusNextControlOnAutoTab')
    }
    let range = document.createRange()
    var sel = document.getSelection()!
    const { startPosition, endPosition } = this.getCursorPos()
    console.log(this.getCursorPos().endPosition)
    range.setStart(this.editableTextBoxRef.childNodes[0], 1)
    // range.setEnd(this.editableTextBoxRef.childNodes[0], endPosition - 1)
    range.collapse(true)
    sel.removeAllRanges()
    sel.addRange(range)
  }
  /**
   * @description updates the dataModel textBox object properties when user insert/delete text
   * inside textBox, updates text and values properties of textBox
   * @function textAndValueUpdate
   * @param event its of type InputEvent
   * @event input
   *
   */

  updateValueProp (event: InputEvent) {
    const controlProp = this.properties
    const text = this.editableTextBoxRef.innerText
    if (controlProp.MaxLength !== 0 && text.length > controlProp.MaxLength!) {
      this.editableTextBoxRef.innerText = controlProp.Value!.toString()
      event.preventDefault()
    } else {
      controlProp.PasswordChar === ''
        ? this.textAndValueUpdate(text)
        : this.handlePasswordChar(event)
    }
  }

  textAndValueUpdate (updateValue: string) {
    const controlPropData = this.properties
    this.isUpdatingText = true
    if (controlPropData.AutoTab && controlPropData.MaxLength! > 0) {
      if (updateValue.length === controlPropData.MaxLength) {
        EventBus.$emit('focusNextControlOnAutoTab')
      }
    }
    if (this.properties.AutoSize) {
      this.updateAutoSize()
    }
    // if (!controlPropData.MultiLine) {
    //   const newLineLength = updateValue.split('\n')
    //   if (newLineLength.length > 0) {
    //     this.editableTextBoxRef.innerText = newLineLength[0]
    //   }
    //   this.setControlCaretPosition(false)
    // }
    this.updateDataModel({
      propertyName: 'Value',
      value: updateValue
    })
    this.updateDataModel({
      propertyName: 'Text',
      value: updateValue
    })
    if (this.properties.ControlSource !== '') {
      this.updateDataModelExtraData({
        propertyName: 'ControlSourceValue',
        value: updateValue
      })
    } else {
      return undefined
    }
  }

  @Watch('properties.Font.FontSize', { deep: true })
  autoSizeValidateOnFontChange () {
    if (this.properties.AutoSize) {
      this.updateAutoSize()
    }
  }

  @Watch('properties.WordWrap', { deep: true })
  autoSizeValidateOnWordWrapChange () {
    if (this.properties.AutoSize) {
      this.updateAutoSize()
    }
  }

  /**
   * @description dragBehavior - if true when dragging
   *  if false the cursor remains in same place
   * @function dragBehavior
   * @param event its of type KeyboardEvent
   * @event dragStart
   */
  dragBehavior (e: Event) {
    if (this.properties.DragBehavior) {
      return true
    }
    e.preventDefault()
  }
  setControlCaretPosition (isFocused: boolean) {
    console.log(this.getCursorPos())
    Vue.nextTick(() => {
      (this.editableTextBoxRef as HTMLSpanElement).focus()
      const el = this.editableTextBoxRef
      const range = document.createRange()
      const sel = window.getSelection()!
      if (el.childNodes[0]) {
        if (isFocused) {
          range.setStart(
            el.childNodes[0],
            this.getCursorPos().endPosition
          )
        } else {
          range.setStart(
            el.childNodes[0],
          this.properties.Value!.toString().length
          )
        }
        range.collapse(true)
        sel.removeAllRanges()
        sel.addRange(range)
      }
    })
  }
  setCursorStartEndPosition (event: MouseEvent) {
    this.openTextContextMenu(event)
    this.handleDelete(event)
  }
  @Watch('isEditMode')
  editModeValidation () {
    if (this.properties.AutoSize && !this.isEditMode) {
      this.updateAutoSize()
    }
    if (this.editableTextBoxRef) {
      this.originalText = this.editableTextBoxRef.innerText
      this.trimmedText = this.originalText.replace(/(\r\n|\n|\r)/gm, ',')
    }
    if (this.isEditMode) {
      Vue.nextTick(() => {
        this.editableTextBoxRef.innerText = this.passwordCharFilter()
        this.setControlCaretPosition(false)
      })
    }
  }

  @Watch('properties.MultiLine')
  multiLineValidate () {
    // debugger
    if (this.editableTextBoxRef) {
      this.originalText = this.editableTextBoxRef.innerText
      this.trimmedText = this.originalText.replace(/(\r\n|\n|\r)/gm, '¶')
      if (this.properties.MultiLine) {
        this.trimmedText = this.originalText.replace(/¶/g, '\n')
        this.editableTextBoxRef.innerText = this.trimmedText
        if (this.properties.PasswordChar === '') {
          this.updateDataModel({
            propertyName: 'Value',
            value: this.trimmedText
          })
        }
      } else {
        this.editableTextBoxRef.innerText = this.trimmedText
        if (this.properties.PasswordChar === '') {
          this.updateDataModel({
            propertyName: 'Value',
            value: this.trimmedText
          })
        }
      }
    }
    if (this.properties.AutoSize) {
      this.updateAutoSize()
    }
  }

  @Watch('properties.AutoSize')
  setAutoHeightWidth () {
    if (this.properties.AutoSize) {
      this.updateAutoSize()
    }
  }
  /**
   * @override
   */
  updateAutoSize () {
    if (this.properties.AutoSize === true) {
      this.$nextTick(() => {
        const textareaRef: HTMLDivElement = this.textareaRef
        this.updateDataModel({
          propertyName: 'Width',
          value: this.editableTextBoxRef.offsetWidth + 2
        })
        this.updateDataModel({
          propertyName: 'Height',
          value: this.editableTextBoxRef.offsetHeight + 5
        })
        if (this.properties.Value === '') {
          this.updateDataModel({
            propertyName: 'Height',
            value: 16
          })
          this.updateDataModel({
            propertyName: 'Width',
            value: 13
          })
        }
      })
    } else {
      return undefined
    }
  }

  created () {
    const propData: controlData = this.data
    if (propData.properties.ControlSource !== '') {
      const controlSourceValue = propData.extraDatas!.ControlSourceValue
      this.updateDataModel({
        propertyName: 'Value',
        value: controlSourceValue
      })
      this.updateDataModel({ propertyName: 'Text', value: controlSourceValue })
    }
    EventBus.$on('updateText', (updateValue: string) => {
      this.editableTextBoxRef.innerText = updateValue
      this.setControlCaretPosition(true)
    })
  }

  /**
   * @description watches ControlSource property
   * @function updateControlSourceValue
   * @param oldVal previous string value
   * @param newVal  new/changed string value
   */
  @Watch('properties.ControlSource', { deep: true })
  updateControlSourceValue (newVal: string, oldVal: string) {
    const propData: controlData = this.data
    if (propData.properties.ControlSource !== '') {
      const controlSourceValue = propData.extraDatas!.ControlSourceValue
      this.updateDataModel({
        propertyName: 'Value',
        value: controlSourceValue
      })
      this.updateDataModel({ propertyName: 'Text', value: controlSourceValue })
    } else {
      this.updateDataModel({
        propertyName: 'Value',
        value: ''
      })
      this.updateDataModel({ propertyName: 'Text', value: '' })
    }
  }
  /**
   * @description keep tracks of key press and selectionStart and selectionEnd
   * updates extra property CursorStartPosition and CursorEndPosition which is required
   * when user insert, update or delete text in textBox
   * @function handleDelete
   * @param event it is of type KeyboardEvent
   * @event keydown
   */
  handleDelete (event: KeyboardEvent | MouseEvent) {
    // debugger
    if (event.target instanceof HTMLSpanElement) {
      let eventType = ''
      if (event instanceof KeyboardEvent && (event.keyCode === 8 || event.keyCode === 46 || (event.ctrlKey && event.code === 'KeyX') || (event.ctrlKey && event.code === 'KeyV') || (event.ctrlKey && event.code === 'KeyA'))) {
        eventType = 'keyboard'
      } else if (event instanceof MouseEvent && event.which === 3) {
        eventType = 'mouse'
      }
      if (eventType) {
        const { startPosition, endPosition } = this.getCursorPos()
        this.updateDataModel({
          propertyName: 'CursorStartPosition',
          value: startPosition!
        })
        this.updateDataModel({
          propertyName: 'CursorEndPosition',
          value: endPosition!
        })
      } else {
        return undefined
      }
    } else {
      throw new Error(
        'Expected HTMLTextAreaElement but found different element'
      )
    }
  }
  /**
   * @description  show selection when TextBox loses focus
   * when HideSelection is false selection is show if user selects any text
   * @function handleBlur
   * @event blur
   *
   */
  handleBlur (
    event: TextEvent
  ) {
    // debugger
    const { startPosition, endPosition } = this.getCursorPos()
    this.getSelectionStart = startPosition
    this.getSelectionEnd = endPosition
    // const s = window.getSelection()!
    // if (s.rangeCount >= 1) {
    //   for (var i = 0; i < s.rangeCount; i++) {
    //     console.log('a')
    //     s.removeRange(s.getRangeAt(i))
    //   }
    // }
    // if (!this.properties.HideSelection) {
    //   if (event.target instanceof HTMLTextAreaElement) {
    //     const eventTarget = event.target
    //     hideSelectionDiv.style.display = 'block'
    //     hideSelectionDiv.style.height = this.properties.Height! + 'px'
    //     hideSelectionDiv.style.width = this.properties.Width! + 'px'
    //     textareaRef.style.display = 'none'
    //     let textarea = eventTarget.value
    //     let firstPart =
    //     textarea.slice(0, eventTarget.selectionEnd) +
    //     '</span>' +
    //     textarea.slice(eventTarget.selectionEnd + Math.abs(0))
    //     let text =
    //     firstPart.slice(0, eventTarget.selectionStart) +
    //     "<span style='background-color:lightblue'>" +
    //     firstPart.slice(eventTarget.selectionStart + Math.abs(0))
    //     hideSelectionDiv.children[0].textContent = text
    //     const divScrollTop = hideSelectionDiv.scrollTop
    //     if (this.textareaScrollTop === 0) {
    //       hideSelectionDiv.scrollTop = divScrollTop
    //     } else {
    //       hideSelectionDiv.scrollTop = this.textareaScrollTop
    //     }
    // } else {
    //   throw new Error('Expected HTMLTextAreaElement but found different element')
    // }
    // } else {
    //   return undefined
    // }
  }
  mounted () {
    this.$el.focus()
  }
  releaseEditMode (event: KeyboardEvent) {
    this.$el.focus()
    this.setContentEditable(event, false)
  }
  eventStoppers () {
    const eventStop = (event: Event) => event.stopPropagation()
    return this.isEditMode === false
      ? null
      : {
        keydown: eventStop
      }
  }
  @Watch('properties.TextAlign')
  textAlignValidate () {
    if (this.properties.AutoSize) {
      this.updateAutoSize()
    }
  }
  @Watch('properties.ScrollBars')
  scrollBarValidate () {
    if (this.properties.AutoSize) {
      this.updateAutoSize()
    }
  }
  @Watch('properties.Enabled')
  enabledValidate () {
    if (this.properties.AutoSize) {
      this.updateAutoSize()
    }
  }
  @Watch('properties.SelectionMargin')
  selectionMarginValidate () {
    if (this.properties.AutoSize) {
      this.updateAutoSize()
    }
  }
  @Watch('properties.Value')
  updateSpanInnerText () {
    if (!this.isUpdatingText) {
      this.editableTextBoxRef.innerText = this.passwordCharFilter()
      this.setControlCaretPosition(true)
      this.isUpdatingText = false
    } else {
      this.isUpdatingText = false
    }
    const propData: controlProperties = this.properties
    if (propData.Text !== propData.Value && propData.ControlSource === '') {
      this.updateDataModel({ propertyName: 'Text',
        value: this.properties.Value! })
    }
  }
  @Watch('properties.PasswordChar')
  updateInnerTextWhenPasswordCharUpdated () {
    this.editableTextBoxRef.innerText = this.passwordCharFilter()
  }
  passwordCharFilter () {
    // debugger
    const controlProp = this.properties
    if (controlProp.PasswordChar !== '' && controlProp.Value !== '') {
      let filteredValue: string = ''
      for (let index = 0; index < controlProp.Value!.toString().length; index++) {
        filteredValue = filteredValue + controlProp.PasswordChar![0]
      }
      return filteredValue
    }
    return controlProp.Value!.toString()
  }
  textBoxClick (event: MouseEvent) {
    if (this.toolBoxSelectControl === 'Select') {
      event.stopPropagation()
    }
    Vue.nextTick(() => {
      if (this.isEditMode) {
        this.editableTextBoxRef.focus()
      }
    })
  }
  getCursorPos () {
    let startPosition = 0
    let endPosition = 0
    const isSupported = typeof window.getSelection !== 'undefined'
    if (isSupported) {
      const selection = window.getSelection()!
      if (selection.anchorOffset > selection.focusOffset) {
        startPosition = selection.focusOffset
        endPosition = selection.anchorOffset
      } else if (selection.anchorOffset === selection.focusOffset) {
        startPosition = selection.focusOffset
        endPosition = selection.focusOffset
      } else {
        startPosition = selection.anchorOffset
        endPosition = selection.focusOffset
      }
    }
    return { startPosition: startPosition, endPosition: endPosition }
  }
  closeTextMenu () {
    EventBus.$emit('closeMenu')
  }
}
</script>

<style scoped>
.text-box-design {
  width: 0px;
  height: 0px;
  left: 0px;
  top: 0px;
  resize: none;
  overflow: hidden;
  box-sizing: border-box;
  caret-color: black;
}
.text-box-design:focus {
  outline: none;
}
.text-box-design::selection {
  background: lightblue;
}
</style>
