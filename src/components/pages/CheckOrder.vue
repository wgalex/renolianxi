<template lang="pug">
  div
    loading(:active.sync='isLoading', :opacity='0.85')
      img(src='@/assets/loading.gif', alt='', srcset='')
      vue-typed-js.justify-content-center.align-items-center(:strings="['波利加載中…']")
        small.font-weight-normal.typing
    .container.p-top
      h3.text-center.my-2 結帳確認
      .my-5.row.justify-content-center
        form.col-md-6(@submit.prevent='payOrder()')
          table.table
            thead
              tr
                th 品名
                th 數量
                th 單價
            tbody
              tr(v-for='item in order.products', :key='item.id')
                td.align-middle {{ item.product.title }}
                td.align-middle {{ item.qty }}/{{ item.product.unit }}
                td.align-middle.text-right {{ item.final_total }}
            tfoot
              tr
                td.text-right(colspan='2') 總計
                td.text-right {{ order.total }}
          table.table
            tbody
              tr
                th(width='100') Email
                td {{ order.user.email }}
              tr
                th 姓名
                td {{ order.user.name }}
              tr
                th 收件人電話
                td {{ order.user.tel }}
              tr
                th 收件人地址
                td {{ order.user.address }}
              tr
                th 付款狀態
                td
                  span(v-if='!order.is_paid') 尚未付款
                  span.text-success(v-else='') 付款完成
          .text-right(v-if='order.is_paid === false')
            button.btn.btn-danger 確認付款去
</template>

<style lang="scss">
  .p-top {
    padding-top: 91px;
  }
</style>

<script>
export default {
  data() {
    return {
      orderId: '',
      order: {
        user: {},
      },
      isLoading: false,
    };
  },
  methods: {
    getOrder() {
      const vm = this;
      const url = `${process.env.APIPATH}/api/${
        process.env.COUSTOMPATH
      }/order/${vm.orderId}`;
      vm.isLoading = true;
      vm.$http.get(url).then((response) => {
        if (response.data.success) {
          vm.order = response.data.order;
          vm.isLoading = false;
        } else {
          vm.isLoading = false;
          vm.$bus.$emit('message:push',
            `出現錯誤惹，好糗Σ( ° △ °|||)︴
            ${response.data.message}`
            , 'danger');
        }
      });
    },
    payOrder() {
      const vm = this;
      const url = `${process.env.APIPATH}/api/${
        process.env.COUSTOMPATH
      }/pay/${vm.orderId}`;
      vm.isLoading = true;
      vm.$http.post(url).then((response) => {
        if (response.data.success) {
          vm.isLoading = false;
          vm.getOrder();
          this.$bus.$emit('message:push',
            '感謝你的購買(*ゝ∀･)v'
            , 'success');
        } else {
          vm.isLoading = false;
          vm.$bus.$emit('message:push',
            `出現錯誤惹，好糗Σ( ° △ °|||)︴
            ${response.data.message}`
            , 'danger');
        }
      });
    },
  },
  created() {
    this.orderId = this.$route.params.orderId;
    this.getOrder();
  },
};
</script>
