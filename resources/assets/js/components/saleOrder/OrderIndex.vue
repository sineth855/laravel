<template>
  <div id="pageTable">
    <v-container grid-list-xl fluid>
      <!--toast message-->
      <message
        ref="child"
        :notifyStatus="notifyStatus"
        :message="message" > 
      ></message><!--#End toast message-->
      <!--#Dialog Message-->
      <dialog-confirm 
        ref="child"
        v-on:event_child="eventChild">
      </dialog-confirm><!--#End Dialog Message-->

      <v-layout row wrap>
        <!--<v-flex sm12>
          <h3>Category Index</h3>
        </v-flex>-->
        <v-flex lg12>
          <v-card>
            <v-toolbar card color="white">
              <v-text-field
              flat
              solo
              prepend-icon="search"
              placeholder="Type something"
              v-model="search"
              hide-details
              class="hidden-sm-and-down"
              ></v-text-field>     
              <v-btn icon>
                <v-icon>filter_list</v-icon>
              </v-btn>
              <v-btn @click="createNew()" color="primary">
                <v-icon>add</v-icon> Create New
              </v-btn>
            </v-toolbar>
            <v-divider></v-divider>
            
            <v-card-text class="pa-0">
              <v-data-table
                :headers="complex.headers"
                :search="search"
                :items="items"
                :rows-per-page-items="[20,50,100,{text:'All','value':-1}]"
                class="elevation-1"
                item-key="name"
                select-all
                v-model="complex.selected"
                >
                <template slot="items" slot-scope="props">
                <td>
                  <v-checkbox
                    primary
                    hide-details
                    v-model="props.selected"
                  ></v-checkbox>
                </td> 
                  <td>{{ props.index+1 }}</td>
                  <td>{{ props.item.invoice_prefix }}</td>
                  <td>{{ props.item.store_name }}</td>
                  <td>{{ props.item.payment_firstname }} {{ props.item.payment_lastname }}</td>
                  <td>{{ props.item.email }}</td>
                  <td>{{ props.item.telephone }}</td>
                  <td>
                    <v-chip label small :color="getColorByStatus('success')" text-color="white" >{{ props.item.status }}</v-chip>
                    <!--<v-chip v-else label small :color="getColorByStatus('warning')" text-color="white" >{{ "Inactive" }}</v-chip>-->
                  </td>
                  <td>
                    <v-btn @click="viewEdit(props.item.order_id)" depressed outline icon fab dark color="primary" small>
                      <v-icon>search</v-icon>
                    </v-btn>
                  </td>
                </template>
              </v-data-table>
            </v-card-text>
          </v-card>
        </v-flex>
      </v-layout>
    </v-container>
  </div>
</template>

<script>

  import {fetchList,updateData,deleteData} from '~/api/saleOrder/order'
  import Message from '~/components/common/Message'
  import DialogConfirm from '~/components/common/DialogConfirm'
  import Flash from '~/services/flash'
  export default {
    data () {
      return {
        routeName: 'SaleOrderForm',
        routeUrl: '/sale_order/order',
        search: '',
        items: [],
        notifyStatus: '',
        message: '',
        flash: Flash.state,
        dialog:false,
        colors: {
          warning: 'blue',
          danger: 'red',
          success: 'green'
        },
        complex: {
          selected: [],
          headers: [
            {
              text: 'ID',
              value: 'id'
            },
            {
              text: 'Invoice No',
              value: 'invoice_prefix'
            },
            {
              text: 'Store',
              value: 'store'
            },
            {
              text: 'Customer',
              value: 'fullName'
            },
            {
              text: 'Email',
              value: 'email'
            },
            {
              text: 'Telephone',
              value: 'telephone'
            },
            {
              text: 'Order Status',
              value: 'order_status_id'
            },
            {
              text: 'Action',
              value: ''
            },
          ]
        }
      };
    },
    components: {
      Message,
      DialogConfirm
    },
    created(){
      Flash.setLoading(true)
      this.getData()
    },
    methods:{
      getColorByStatus (status) {
        return this.colors[status];
      },
      viewEdit(id){
        this.$router.push(this.routeUrl+'/'+id)
      },
      createNew(){
        this.$router.push({name:this.routeName})
      },
      eventChild: function(_dialogMessage,dataId) {
        if(_dialogMessage=='yes'){
          deleteData(dataId).then(response => {
            if(response.success == true){
              this.notifyStatus = 'success'
              this.message = response.message
              this.getData()
            }else{
              this.notifyStatus = 'error'
              this.message = response.message
            }
          })
        }
      },
      btnConfirm(id){
        this.dialog = true
        this.$refs.child.child_method(this.dialog,id)
      },
      getData() {
        fetchList().then(response => {
          Flash.setLoading(false)
          this.items = response['data']
        })
      }
    }
  }
</script>
