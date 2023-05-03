<template>
<div class="home">
    <v-container>
        <div class="box">
          <img src="https://kalbenutritionals.com/images/kalbe-nutritionals-logo.png" alt="" class="logo-brand">
            <div class="content" v-if="customerShow">
                <div class="text-center">
                    <h2>Pilih Customer</h2>
                </div>
                <v-autocomplete v-model="customer" :items="customerList" item-text="txtCustomerName" item-value="intCustomerID" clearable return-object></v-autocomplete>
                <v-btn :disabled="!customer" @click="customerShow = false, productShow = true">Pilih Customer</v-btn>
            </div>
            <div class="content" v-if="productShow">
                <div class="text-center">
                    <h2>Pilih Produk</h2>
                </div>
                <v-autocomplete v-model="product" :items="productList" item-text="txtProductName" item-value="intProductID" placeholder="Pilih Produk" clearable return-object persistent-hint :hint="`Qty: ${product?.intQty ? product.intQty : '0'}`"></v-autocomplete>
                <v-text-field label="Quantity" placeholder="Quantity" v-model="quantity"></v-text-field>
                <v-btn :disabled="!product || !quantity" @click="orderProduct()" :loading="isLoading">Beli Product</v-btn>
            </div>
            <div class="content" v-if="resultShow">
                <div class="text-center">
                    <h2>Terima Kasih</h2>
                    <p>Detail Pesanan</p>
                </div>
                <table>
                    <tr>
                        <td>Nama :</td>
                        <td>{{customer?.txtCustomerName}}</td>
                    </tr>
                    <tr>
                        <td>Product :</td>
                        <td>{{product?.txtProductName}}</td>
                    </tr>
                    <tr>
                        <td>Quantity :</td>
                        <td>{{quantity}}</td>
                    </tr>
                </table>
                <v-btn @click="orderView()">Lihat Pesanan</v-btn>
            </div>
            <div class="content" v-if="orderShow">
                <div class="text-center">
                    <h2>Pesanan Kamu</h2>
                </div>
                <table>
                    <tr>
                        <th>Produk</th>
                        <th>Qty</th>
                        <th>Total</th>
                    </tr>
                    <tr v-for="item,idx in orderList" :key="idx">
                        <td>{{item.product.txtProductName}}</td>
                        <td>{{item.intQty}}</td>
                        <td>{{ item.totalPrice }}</td>
                    </tr>
                </table>
                <v-btn @click="repeatOrder()">Pesan Lagi</v-btn>
            </div>
        </div>
    </v-container>
</div>
</template>

<script>
// @ is an alias to /src
import HelloWorld from '@/components/HelloWorld.vue'
import axios from 'axios';
export default {
    name: 'HomeView',
    components: {
        HelloWorld
    },
    data() {
        return {
            customerShow: true,
            customer: null,
            customerList: [],

            resultShow: false,
            productShow: false,
            product: null,
            productList: [],
            quantity: null,
            isLoading: false,
            orderList: [],
            orderShow: false,
        }
    },
    methods: {
        getCustomer() {
            axios.get('http://localhost:3000/api/v1/customer').then(response => {
                this.customerList = response.data.data.rows;
            })
        },
        getProduct() {
            axios.get('http://localhost:3000/api/v1/product').then(response => {
                this.productList = response.data.data.rows;
            })
        },
        getCustomerOrder() {
            axios.get(`http://localhost:3000/api/v1/order?id=${this.customer.intCustomerID}`).then(response => {
                this.orderList = response.data.data.rows;
            })
        },
        orderProduct() {
            this.isLoading = true;
            axios.post('http://localhost:3000/api/v1/order', {
                intCustomerID: this.customer.intCustomerID,
                intProductID: this.product.intProductID,
                intQty: this.quantity
            }).then(response => {
                console.log(response);
                this.isLoading = false;
                this.resultShow = true;
                this.productShow = false;
            }).catch(error => {
                console.log(error);
            })
        },
        orderView() {
            this.getCustomerOrder();
            this.orderShow = true;
            this.resultShow = false;
        },
        repeatOrder() {
            this.productShow = true;
            this.orderShow = false;
            this.product = null;
            this.quantity = null;

        }
    },
    mounted() {
        this.getCustomer();
        this.getProduct();
    }
}
</script>

<style lang="scss" scoped>

.logo-brand {
  position: absolute;
  top: 70px;
  left: 50%;
  transform: translateX(-50%);
}
.box {
    height: calc(100vh - 50px);
    width: 100%;
    display: flex;
    align-items: center;
    justify-content: center;

    .content {
        border-radius: 8px;
        border: 1px solid #ccc;
        padding: 40px;
        box-shadow: 0px 0px 10px #ccc;
        width: 400px;

        table {
            width: 100%;
            margin-bottom: 16px;
            text-align: left;
        }
    }
}
</style>
