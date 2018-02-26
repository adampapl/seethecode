<!-- parent available name : local variable -->
<template>
    <div class='componentInvoiceDetail' >
        <LoadingBox :isLoading="isLoading"></LoadingBox>
        <div v-if='!isLoading && vat_types.length==0'>
            <p>{{ trans('invoice.own_company_details_missing') }}</p>
            <p>
                <router-link to='/settings/own-company' class="btn btn-primary">
                    {{ trans('invoice.own_company_details_missing_btn') }}
                </router-link>
            </p>
        </div>
        <form v-if='!isLoading && vat_types.length > 0'>
            <div class='form-group'>
                <div class='pull-left'>
                    <h4>{{trans('invoice.number')}}:</h4>
                    {{invoice.number}}
                </div>
                <div class='pull-right'>
                    <h4>{{ trans('invoice.date')}}:</h4>
                    <datepicker v-model="invoice.date" :allowedDateRange="allowedDateRange" position='left'></datepicker>
                    <form-error :errors="vErrors" field="invoice.date"></form-error>
                </div>
                <div class="clearfix"></div>
            </div>

            <div v-if="invoice.client_id==0 || typeof invoice.client_id =='undefined' ">
                <h4>{{ trans('invoice.select_client')}}:</h4>
                <div class="col-xs-12 col-sm-10 col-md-8 col-lg-6">
                    <autocomplete-clients @selected="clientSelected"></autocomplete-clients>
                </div>
            </div>

            <div v-if="invoice.client_id>0" class="invoice-infoblock">
                <h4>
                    {{trans('invoice.billed_to')}}:
                </h4>
                <p>
                    {{client.name}} - {{client.owner_name}} {{client.owner_surname}}
                    <button type="button" class="btn btn-info btn-sm btn-icon bottom15 right15" @click.prevent='clientReset'>
                        <span>{{trans('invoice.change_client_btn')}}</span>
                    </button>
                </p>
                <address>
                    <span>
                        {{client.address_street}} {{client.address_house}}<br/>
                        {{client.address_postcode}} {{client.address_city}}<br/>
                        {{clientCountryName}}<br/>
                        {{trans('client.reg_number_short')}}: {{client.reg_number}}<br/>
                        {{trans('client.tax_code_short')}}: {{client.tax_code}}<br/>
                    </span>
                </address>
            </div>

            <form-error :errors="vErrors" field="invoice.client_id"></form-error>

            <br/><br/>
            <div class="clearfix"></div>
            <div v-show="$store.state.settings.invoiceSettings.useBtwReverseCharge">
                <div class="row">
                    <div class="col-xs-12">
                        <div class="form-group">
                            <div class="checkbox">
                                <label>
                                    <input type="checkbox" v-model="invoice.isReverseCharge" />
                                    {{trans('invoice.isBtwReverseCharge')}}
                                </label>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <h4>{{trans('invoice.invoice_lines_list_title')}}:</h4>
            <form-error :errors="vErrors" field="invoiceLines"></form-error>
            <table class='table notable'>
                <thead>
                <tr>
                    <th></th>
                    <th>{{trans('invoiceLines.name')}}</th>
                    <th>{{trans('invoiceLines.count')}}</th>
                    <th>{{trans('invoiceLines.unit')}}</th>
                    <th>{{trans('invoiceLines.unit_cost')}}</th>
                    <th v-show="isShowVatColumn">{{trans('invoiceLines.vat')}}</th>
                    <th></th>
                </tr>
                </thead>
                <tbody>
                <tr v-for="(invoiceLine, invoiceLineIndex) in invoiceLines">
                    <td >
                        <button type="button" class="btn btn-info" @click.prevent='removeLine(invoiceLineIndex)' style='width:100%'><i class="fa fa-remove"></i></button>
                    </td>
                    <td :data-label="trans('invoiceLines.name')">
                        <form-error :errors="vErrors" :field="'invoiceLines.'+invoiceLineIndex+'.name'"></form-error>
                        <input type='text' v-model='invoiceLine.name' style='width:100%' class='form-control' maxlength="50" />
                    </td>
                    <td :data-label="trans('invoiceLines.count')">
                        <form-error :errors="vErrors" :field="'invoiceLines.'+invoiceLineIndex+'.count'"></form-error>
                        <input type='text' v-model.number='invoiceLine.count' class='form-control count' />
                    </td>
                    <td :data-label="trans('invoiceLines.unit')">
                        <form-error :errors="vErrors" :field="'invoiceLines.'+invoiceLineIndex+'.unit_type'"></form-error>
                        <select v-model='invoiceLine.unit_type' class='form-control' >
                            <option v-for="unit_type in unit_types" v-bind:value='unit_type.val'>{{unit_type.name}}</option>
                        </select>
                    </td>
                    <td :data-label="trans('invoiceLines.unit_cost')">
                        <form-error :errors="vErrors" :field="'invoiceLines.'+invoiceLineIndex+'.unit_cost'"></form-error>
                        <money v-model.number='invoiceLine.unit_cost' v-bind="moneyFormat" class='form-control unit_cost' ></money>
                    </td>
                    <td :data-label="trans('invoiceLines.vat')" v-show="isShowVatColumn">
                        <form-error :errors="vErrors" :field="'invoiceLines.'+invoiceLineIndex+'.vat'"></form-error>
                        <select v-model.number='invoiceLine.vat' class='form-control' >
                            <option v-for="vat_type in activeVATtypes" v-bind:value="vat_type.value">{{ vat_type.text }}</option>
                        </select>
                    </td>
                    <td>
                        {{trans('invoice.calculated_netto')}}: {{ lineComputed[parseInt(invoiceLineIndex)].netto.formatMoney() }}<br/>
                        <span v-show='invoiceLine.vat>=0 && isShowVatColumn'>{{trans('invoice.calculated_vat')}}: {{ lineComputed[parseInt(invoiceLineIndex)].vat.formatMoney() }}<br/></span>

                        {{trans('invoice.calculated_total')}}: {{ lineComputed[parseInt(invoiceLineIndex)].total.formatMoney() }}
                    </td>
                </tr>
                <tr class="summary">
                    <td :colspan='summaryColspan' data-label=''>
                        <button type="button" class="btn btn-info btn-icon bottom15 right15" @click='addLine'>
                            <i class="fa fa-plus"></i> &nbsp; <span>{{trans('invoice.invoice_line_add_btn')}}</span>
                        </button>
                    </td>
                </tr>

                </tbody>
            </table>
            <table class="table invoiceSummaryTable">
                <tr >
                    <td class="title">Subtotal:</td>
                    <td class="computedSummary">{{summaryComputed.subtotal.formatMoney()}}</td>
                </tr>
                <tr v-if="Vat[0]>=0" v-for="(Vat, i) in sortedVATs">
                    <td class="title">VAT {{Vat[0]}}% ({{Vat[2].formatMoney()}})</td>
                    <td class="computedSummary">{{ Vat[1].formatMoney() }}</td>
                </tr>
                <tr v-if="invoice.isReverseCharge">
                    <td class="title"></td>
                    <td class="computedSummary">{{trans('invoice.lineBtwReverseCharge')}} {{client.tax_code}}</td>
                </tr>
                <tr>
                    <td class="title"><b>Total:</b></td>
                    <td class="computedSummary"><b v-if="this.invoice.total">{{this.invoice.total.formatMoney()}}</b></td>
                </tr>
            </table>
            <h4>{{trans('invoice.notes')}}:</h4>
            <autosize-textarea v-model='invoice.notes' class="form-control" maxlength="127"></autosize-textarea>
            <form-error :errors="vErrors" field="invoice.notes"></form-error>

            <h4>{{trans('invoice.data_paymentDays')}}:</h4>
            <form-error :errors="vErrors" field="invoice.data_paymentDays"></form-error>
            <input type='number' v-model='invoice.data_paymentDays' class='form-control' style='max-width:60px;'/>
            <br/>
            <br/>
            <hr/>
            <button type="button" class="btn btn-primary btn-icon bottom15 right15" @click='saveInvoice(false)'>
                <i class="fa fa-save"></i> &nbsp; <span>{{trans('invoice.invoice_save_btn')}}</span>
            </button>

            <button type='button' class="btn btn-info btn-icon bottom15 right15" @click='downloadPDF' >
                <i class="fa fa-download"></i> &nbsp; <span>{{trans('invoice.invoice_PDF_btn')}}</span>
            </button>

        </form>
    </div>
</template>

<script>
    import {Money} from 'v-money'
    import { EventBus } from './EventBus.vue';
    import helpers from './../helpers.js';
    import LoadingBox from './baseComponents/LoadingBox';
    Vue.component('LoadingBox', LoadingBox);
    import FormError from './FormError.vue';
    Vue.component('form-error', FormError);
    import AutocompleteClients from './AutocompleteClients.vue';
    Vue.component('autocomplete-clients', AutocompleteClients);
    import AutosizeTextarea from './elements/AutosizeTextarea.vue';
    Vue.component('autosize-textarea', AutosizeTextarea);
    import Datepicker from './elements/Datepicker';
    Vue.component('datepicker', Datepicker);
    
    export default {
        components: { Money },
        mounted() {
            this.fetchInvoiceData();
            this.getConfig();

            let self = this;
            self.isLoading++;
            this.$store.dispatch('getMyInvoiceSettings')
                .then(
                    response => {
                        self.isLoading--;
                    },
                    error => {
                        self.isLoading--;
                    }
                )
        },
        props:{
            invoice_id: 0,
        },
        data(){
            return {
                trans:trans,  // global function for translations - trans('msg.test',{name:'Adam'})
                moneyFormat: moneyFormat,
                isLoading: 0,
                invoice: {},
                invoiceLines: [],
                client: {}, // we will get it from autocomplete component
                allowedDateRange:{from:null, to:null},
                vat_types: [],
                unit_types: [],
                vErrors: {}
            }
        },
        computed:{
            clientCountryName(){
                if(this.client.address_country){
                    return trans('countries.'+this.client.address_country);
                }
                else{
                    return '';
                }
            },
            defaultVatType(){
                let dVat = this.vat_types.filter(vat => vat.is_default==1);
                /* get default VAT or first found VAT or null */
                return _.get(dVat,'0.value',
                    _.get(this.vat_types, '0.value', null)
                );
            },
            activeVATtypes(){
                let self = this;
                return this.vat_types.filter(vat => (vat.date_start==null || vat.date_start <= self.invoice.date) && (vat.date_end==null || vat.date_end >= self.invoice.date) );
            },
            isShowVatColumn(){
                return ! this.invoice.isReverseCharge;
            },
            summaryColspan(){
                return this.invoice.isReverseCharge ? 4 : 5;
            },
            lineComputed(){
                return this.computeInvoiceLines();
            },
            summaryComputed(){
                let subtotal = 0;
                let VATs = []; // [[vat%, vatAmount,nettoAmount], ...]

                let invoiceTotal = 0;
                this.invoiceLines.map((line, index) => {
                    if( this.invoice.isReverseCharge ){
                        line.vat = 0;
                    }
                    let lineNetto = line.unit_cost * line.count;
                    let lineVat = (line.vat>=0) ? lineNetto * line.vat / 100 : 0;
                    let lineTotal = helpers.round(lineNetto + lineVat,2);
                    invoiceTotal += lineTotal;
                    subtotal += lineNetto;
                    if(this.isShowVatColumn){
                        VATs = this._updateVAT(VATs, line.vat, lineVat, lineNetto);
                    }
                });
                this.invoice.total = invoiceTotal;
                return {subtotal: subtotal, VATs: VATs};
            },
            sortedVATs(){
                return this.summaryComputed.VATs.sort((a, b) => {
                    let x = a[0]; let y = b[0];
                    return ((x < y) ? -1 : ((x > y) ? 1 : 0));
                });
            }
        },
        methods: {
            clientSelected(client){
                this.invoice.client_id = client.id;
                this.client = client;
                this.$forceUpdate();
            },
            clientReset(){
                this.invoice.client_id = 0;
                this.client = {};
                this.$forceUpdate();
            },
            fetchInvoiceData(){
                this.isLoading++;
                let self = this;
                axios.get('/api/invoices/'+this.invoice_id)
                    .then( rsp => {
                        self.invoice.total = 0;
                        self.invoice = rsp.data.invoice;
                        self.allowedDateRange = rsp.data.allowedDateRange;
                        self.invoiceLines = rsp.data.invoiceLines;
                        self.client = rsp.data.client;

                        if(self.invoice_id==0 && self.invoiceLines.length==0){
                            self.addLine();
                        }
                        self.isLoading--;
                    })
                    .catch( error => {
                        console.log(error);
                        self.isLoading--;
                    });
            },
            getConfig(){
                let self = this;
                self.isLoading++;
                axios.get('/api/config',{params: { unit_types:{},vat_types:{} }})
                    .then( rsp => {
                        self.unit_types = rsp.data.unit_types;
                        self.vat_types = rsp.data.vat_types;
                        self.isLoading--;
                    })
                    .catch( error => {
                        console.log(error);
                        self.isLoading--;
                    });
            },
            addLine(){
                this.invoiceLines.push( this.defaultInvoiceLine() );
            },
            removeLine(index){
                this.invoiceLines.splice( index, 1 );
            },
            saveInvoice(andDownloadPDF){
                let self = this;
                axios.post(
                    '/api/invoices/store',
                    {
                        invoice: this.invoice,
                        invoiceLines: this.invoiceLines
                    })
                    .then( response => {
                        self.vErrors = {};
                        if(andDownloadPDF){
                            window.location = '/api/pdf/download/invoice?id=' + response.data.invoice_id + '&token=' + localStorage.getItem('id_token');
                        }
                        EventBus.$emit('invoices.changed', self.invoice);
                        self.$emit('close');
                    })
                    .catch( (error, response) => {
                        /*
                        {"invoice.client_id":["The invoice.client id field is required."],"invoiceLines.0.name":["The invoiceLines.0.name field is required."],"invoiceLines.0.unit_cost":["The invoiceLines.0.unit_cost field is required."],"invoiceLines.0.count":["The invoiceLines.0.count field is required."],"invoiceLines.0.vat":["The selected invoiceLines.0.vat is invalid."]}
                        */
                        if(error.response.status==422){
                            console.log(error.response.data);
                            self.vErrors = error.response.data;
                            helpers.scrollToError();
                        }
                    });
            },
            downloadPDF(){
                this.saveInvoice( true );
            },
            computeInvoiceLines(){
                let c = {};
                this.invoiceLines.map( (line, index) => {
                    c[index] = {};
                    let lineNetto = line.unit_cost * line.count;
                    let lineVat = (line.vat>=0) ? lineNetto * line.vat / 100 : 0;
                    let lineTotal = helpers.round(lineNetto + lineVat,2);

                    c[index].netto = helpers.isNaN( helpers.round(lineNetto,2), '--');
                    c[index].vat = helpers.isNaN( helpers.round(lineVat,2), '--');
                    c[index].total = helpers.isNaN( helpers.round(lineTotal,2), '--');
                });
                return c;
            },
            defaultInvoiceLine(){
                return {
                    name: '',
                    unit_type: 'piece',
                    unit_cost: 0,
                    count: 0,
                    vat: this.defaultVatType
                }
            },
            
            _updateVAT(VATs, percent, amountVAT, amountNetto){
                let found = false;
                VATs.map( (Vat,i) => {
                    if (Vat[0] == percent){
                        VATs[i][1] += amountVAT;
                        VATs[i][2] += amountNetto;
                        found = true;
                    }
                });
                if (found == false){
                    VATs.push( [percent, amountVAT, amountNetto] );
                }
                return VATs;
            },
        }
    }
</script>