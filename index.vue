<template>
  <div>
    <input v-model="model"
      v-show="isPhone"
      ref="phoneLogin"
      @keydown="keyPressHandler($event)"
      @input="updateValue()"
      type="tel"
      placeholder="Email или телефон"
      class="form-control form-control-lg" />
    <input v-model="model"
      v-show="!isPhone"
      ref="emailLogin"
      type="email"
      @keydown="keyPressHandler($event)"
      @input="updateValue()"
      placeholder="Email или телефон"
      class="form-control form-control-lg" />
  </div>
</template>
<script>
export default {
  data() {
    return {
      model: this.value,
      isPhone: this.initialType === 'phone',
      startedWithEight: false
    }
  },
  props: {
    value: {
      default: ''
    },
    initialType: {
      type: String,
      default: 'phone'
    }
  },
  created() {
    if ( this.value.length ) {
      this.checkModel( this.value )
    }
  },
  watch: {
    model( val ) {
      this.checkModel( val )
    }
  },
  methods: {
    checkModel( val ) {
      if ( val && !val.length ) this.startedWithEight = false;
      if ( this.isPhone ) {
        if ( /[^\d]/.test( val ) && !this.isFormatted( val ) ) {
          this.setTypeMail()
        } else {
          this.formatPhone()
        }
      } else if ( /^[\d]+$/.test( val ) ) {
        this.setTypeTel()
      }
    },
    keyPressHandler( ev ) {
      if ( ev.key === 'Backspace' && this.isFormatted( this.model ) ) {
        if ( !window.getSelection().toString().length ) {
          ev.preventDefault();
          this.deleteDigit()
        }
      } else if ( ev.key && /\D/.test( ev.key ) && ev.key.length === 1 && ev.key !== '+' && !ev.metaKey && !ev.ctrlKey ) {
        if ( this.isPhone ) this.model += ev.key
        this.setTypeMail()
      } else if ( ev.key === '+' ) {
        if ( this.isPhone ) ev.preventDefault();
        this.setTypeTel()
      }

    },
    deleteDigit() {
      let digits = this.model.replace( /\D/g, '' ).substr( 1 );
      this.model = digits.substr( 0, digits.length - 1 )
      if ( !digits.length ) {
        this.model = ''
        this.setTypeMail()
      } else {
        this.formatPhone()
      }
    },
    setTypeMail() {
      if ( this.isPhone ) {
        this.isPhone = false
        if ( this.model[ 0 ] === '+' ) this.model = this.model.substr( 2 )
        this.model = this.model.replace( /[\(\)\-_\s]/g, '' );
        if ( this.startedWithEight && this.model.length ) this.model = '8' + this.model
        setTimeout( () => { this.$refs[ 'emailLogin' ].focus() } );
      }
    },
    setTypeTel() {
      if ( !this.isPhone ) {
        this.isPhone = true
        this.formatPhone()
        setTimeout( () => { this.$refs[ 'phoneLogin' ].focus() } )
      }
    },
    updateValue() {
      this.$emit( 'input', this.isPhone ? this.model.replace( /[^\d\+]/g, '' ) : this.model )
    },
    formatPhone() {
      let login;

      if ( this.isFormatted( this.model ) ) {
        login = JSON.parse( JSON.stringify( this.model ) ).replace( /\D/g, '' ).substr( 1 );
      } else {
        login = JSON.parse( JSON.stringify( this.model ) );
      }
      let startEight = this.model === '8';
      if ( startEight ) {
        this.model = '+7';
        login = '+7';
        this.startedWithEight = true;
      }
      while ( login.length < ( startEight ? 12 : 10 ) ) {
        login += '_'
      }
      this.model = `+7 (${login.substr((startEight?2:0), 3)}) ${login.substr((startEight?5:3), 3)}-${login.substr((startEight?8:6), 2)}-${login.substr((startEight?10:8), 2)}`
      this.updateValue()
    },

    isFormatted( str ) {
      return /\+7\s\((\d|_){3}\)\s(\d|_){3}\-(\d|_){2}\-(\d|_){2}/.test( str )
    },
  }
};
</script>
