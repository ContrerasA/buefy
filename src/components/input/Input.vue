<template>
    <div
        class="control"
        :class="rootClasses"
    >
        <input
            v-if="type !== 'textarea'"
            ref="input"
            class="input"
            :class="[inputClasses, customClass]"
            :type="newType"
            :autocomplete="newAutocomplete"
            :maxlength="maxlength"
            :value="computedValue"
            v-bind="$attrs"
            @input="onInput"
            @blur="onBlur"
            @focus="onFocus">

        <textarea
            v-else
            ref="textarea"
            class="textarea"
            :class="[inputClasses, customClass]"
            :maxlength="maxlength"
            :value="computedValue"
            v-bind="$attrs"
            @input="onInput"
            @blur="onBlur"
            @focus="onFocus"/>

        <b-icon
            v-if="icon"
            class="is-left"
            :class="{'is-clickable': iconClickable}"
            :icon="icon"
            :pack="iconPack"
            :size="iconSize"
            @click.native="iconClick"/>

        <b-icon
            v-if="!loading && (passwordReveal || statusTypeIcon)"
            class="is-right"
            :class="{ 'is-clickable': passwordReveal }"
            :icon="passwordReveal ? passwordVisibleIcon : statusTypeIcon"
            :pack="iconPack"
            :size="iconSize"
            :type="!passwordReveal ? statusType : 'is-primary'"
            both
            @click.native="togglePasswordVisibility"/>

        <b-icon
            v-else-if="iconRight"
            class="is-right"
            :class="{ 'is-clickable': iconRightClickable }"
            :icon="iconRight"
            :pack="iconPack"
            :size="iconSize"
            both
            @click.native="$emit('icon-right-click')"/>

        <small
            v-if="maxlength && hasCounter && type !== 'number'"
            class="help counter"
            :class="{ 'is-invisible': !isFocused }">
            {{ valueLength }} / {{ maxlength }}
        </small>
    </div>
</template>

<script>
import Icon from '../icon/Icon'
import config from '../../utils/config'
import FormElementMixin from '../../utils/FormElementMixin'

export default {
    name: 'BInput',
    components: {
        [Icon.name]: Icon
    },
    mixins: [FormElementMixin],
    inheritAttrs: false,
    props: {
        value: [Number, String],
        type: {
            type: String,
            default: 'text'
        },
        passwordReveal: Boolean,
        iconClickable: Boolean,
        hasCounter: {
            type: Boolean,
            default: () => config.defaultInputHasCounter
        },
        customClass: {
            type: String,
            default: ''
        },
        iconRight: String,
        iconRightClickable: Boolean
    },
    data() {
        return {
            newValue: this.value,
            newType: this.type,
            newAutocomplete: this.autocomplete || config.defaultInputAutocomplete,
            isPasswordVisible: false,
            _elementRef: this.type === 'textarea'
                ? 'textarea'
                : 'input'
        }
    },
    computed: {
        computedValue: {
            get() {
                return this.newValue
            },
            set(value) {
                this.newValue = value
                this.$emit('input', value)
                !this.isValid && this.checkHtml5Validity()
            }
        },
        rootClasses() {
            return [
                this.iconPosition,
                this.size,
                {
                    'is-expanded': this.expanded,
                    'is-loading': this.loading,
                    'is-clearfix': !this.hasMessage
                }
            ]
        },
        inputClasses() {
            return [
                this.statusType,
                this.size,
                { 'is-rounded': this.rounded }
            ]
        },
        hasIconRight() {
            return this.passwordReveal || this.loading || this.statusTypeIcon || this.iconRight
        },

        /**
        * Position of the icon or if it's both sides.
        */
        iconPosition() {
            if (this.icon && this.hasIconRight) {
                return 'has-icons-left has-icons-right'
            } else if (!this.icon && this.hasIconRight) {
                return 'has-icons-right'
            } else if (this.icon) {
                return 'has-icons-left'
            }
        },

        /**
        * Icon name (MDI) based on the type.
        */
        statusTypeIcon() {
            switch (this.statusType) {
                case 'is-success': return 'check'
                case 'is-danger': return 'alert-circle'
                case 'is-info': return 'information'
                case 'is-warning': return 'alert'
            }
        },

        /**
        * Check if have any message prop from parent if it's a Field.
        */
        hasMessage() {
            return !!this.statusMessage
        },

        /**
        * Current password-reveal icon name.
        */
        passwordVisibleIcon() {
            return !this.isPasswordVisible ? 'eye' : 'eye-off'
        },
        /**
        * Get value length
        */
        valueLength() {
            if (typeof this.computedValue === 'string') {
                return this.computedValue.length
            } else if (typeof this.computedValue === 'number') {
                return this.computedValue.toString().length
            }
            return 0
        }
    },
    watch: {
        /**
        * When v-model is changed:
        *   1. Set internal value.
        */
        value(value) {
            this.newValue = value
        }
    },
    methods: {
        /**
        * Toggle the visibility of a password-reveal input
        * by changing the type and focus the input right away.
        */
        togglePasswordVisibility() {
            this.isPasswordVisible = !this.isPasswordVisible
            this.newType = this.isPasswordVisible ? 'text' : 'password'

            this.$nextTick(() => {
                this.$refs.input.focus()
            })
        },

        /**
        * Input's 'input' event listener, 'nextTick' is used to prevent event firing
        * before ui update, helps when using masks (Cleavejs and potentially others).
        */
        onInput(event) {
            this.$nextTick(() => {
                if (event.target) {
                    this.computedValue = event.target.value
                }
            })
        },

        iconClick(event) {
            this.$emit('icon-click', event)
            this.$nextTick(() => {
                this.$refs.input.focus()
            })
        }
    }
}
</script>
