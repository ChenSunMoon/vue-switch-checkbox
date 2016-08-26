<style>
    .slider-sm {
        position: absolute;
        cursor: pointer;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        background-color: #ccc;
        -webkit-transition: .4s;
        transition: .4s;
    }
    .switch-sm {
        width: 40px;
        height: 24px;
    }
    .slider-sm:before {
        height: 20px;
        width: 20px;
        position: absolute;
        content: "";
        left: 2px;
        bottom: 2px;
        background-color: white;
        -webkit-transition: .4s;
        transition: .4s;
        box-shadow: 2px 2px 2px #888888;
    }
    input:checked + .slider-sm:before {
        -webkit-transform: translateX(16px);
        -ms-transform: translateX(16px);
        transform: translateX(16px);
    }
    input:checked + .slider-sm {
        border-color: rgb(129, 200, 104);
        box-shadow: rgb(129, 200, 104) 0px 0px 0px 16px inset;
        transition: border 0.4s, box-shadow 0.4s, background-color 1.2s;
        background-color: rgb(129, 200, 104)
    }
    .switch {
        position: relative;
        display: inline-block;
    }

    /* Hide default HTML checkbox */
    .switch input {display:none;}

    input:focus + .slider {
        box-shadow: 0 0 1px #2196F3;
    }

    /* Rounded sliders */
    .slider.round.slider-sm {
        border-radius: 24px;
    }

    .slider.round:before {
        border-radius: 50%;
    }

    .slider-md {
        position: absolute;
        cursor: pointer;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        background-color: #ccc;
        -webkit-transition: .4s;
        transition: .4s;
    }
    .switch-md {
        width: 60px;
        height: 34px;
    }
    .slider-md:before {
        height: 30px;
        width: 30px;
        position: absolute;
        content: "";
        left: 2px;
        bottom: 2px;
        background-color: white;
        -webkit-transition: .4s;
        transition: .4s;
        box-shadow: 2px 2px 2px #888888;
    }
    input:checked + .slider-md:before {
        -webkit-transform: translateX(26px);
        -ms-transform: translateX(26px);
        transform: translateX(26px);
    }
    input:checked + .slider-md {
        border-color: rgb(129, 200, 104);
        box-shadow: rgb(129, 200, 104) 0px 0px 0px 16px inset;
        transition: border 0.4s, box-shadow 0.4s, background-color 1.2s;
        background-color: rgb(129, 200, 104)
    }
    .switch {
        position: relative;
        display: inline-block;
    }

    /* Hide default HTML checkbox */
    .switch input {display:none;}

    input:focus + .slider {
        box-shadow: 0 0 1px #2196F3;
    }

    /* Rounded sliders */
    .slider.round.slider-md {
        border-radius: 34px;
    }

    .slider.round:before {
        border-radius: 50%;
    }

    .error-msg-slider {
        font-size:12px;
        color: #db372d;
    }
    .box-switch {
        float:left;
        margin-right: 10px;
    }
.lista-switches .list-group-item {
padding-bottom:0;
}
</style>
<template>
    <div class="box-switch" v-if="!isLoading">
        <label class="switch switch-sm">
            <input type="checkbox" :class="classe" v-model="checked" @click="setChecked(true)" :disabled="disabled" :name="name" :id="real_id" :checked="checked" data-color="#81c868" data-size="small"/>
            <div class="slider round slider-sm"></div>
        </label>
    </div>
    <div v-if="isLoading">
        <img src="../img/loading.gif" alt="Carregando">
    </div>
    <div v-show="hasError"><p class="error-msg-slider">{{error_msg}}</p></div>
</template>
<script>
    export default {
        ready: function () {
            this.initSwitch();
            this.isLoading = false;
        },
        props: {
            checked: String | Boolean,
            id: String,
            name: String | Number,
            param_url: {},
            disabled: String | Boolean,
            bind_model: {
                field: ''
            }
        },
        data: function () {
            return {
                classe: '',
                options: {
                    size: 'small'
                },
                real_id: this._uid,
                isLoading: true,
                error_msg: ''
            }
        },
        events: {
            modelUpdated: function (model) {

            }
        },
        methods: {
            //DestrÃ³i e inicia switch. limitaÃ§Ã£o do switchery.
            initSwitch: function () {
                this.clearErrorMsg();
            },
            hasError: function () {
                return this.error_msg.length > 0;
            },
            clearErrorMsg: function () {
                this.error_msg = '';
            },
            setDisabled: function (bool) {
                this.disabled = bool;
            },
            setChecked: function (runCallback) {
                var checked = this.checked;
                if (runCallback) {
                    this.runAjaxCallback(checked);
                }
            },
            getSelector: function () {
                return document.getElementById(this.real_id);
            },
            hasCallback: function () {
                // Verifica se objeto param_url tem propriedades
                return Object.keys(this.param_url).length > 0;
            },
            getCallbackUrl: function () {
                var self = this;
                if (self.hasCallback())
                    var params = self.param_url;
                var url = '';

                for (var param in params) {
                    url += params[param] + '/';
                }
                return url.slice(0, -1); // Removendo a ultima / da url.
            },
            setErrorMsg(response, checked){

                if (response.data.msg) {
                    this.$set('error_msg', response.data.msg);
                }
                this.checked = checked;
            },
            runAjaxCallback: function (checked) {
                var self = this;
                if (!this.hasCallback()) {
                    return false;
                }
                this.clearErrorMsg();
                this.setDisabled(true);

                // Altera para novo status
                this.checked = !checked;
                this.$http.post(this.getCallbackUrl(),
                        {
                            value: !checked,
                        }).then(function (response) {

                    if (!response.data.status) {
                        this.setErrorMsg(response, checked);
                    } else{
                      this.$parent.$emit('switch_changed',{checked: this.checked, id:this.id,name: this.name})
                    }
                    this.setDisabled(false);
                }, function (response) {
                    this.setDisabled(false);
                    this.setErrorMsg(response, checked);
                });
            }

        }
    }
    </script>
