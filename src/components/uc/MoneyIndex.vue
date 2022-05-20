<template>
  <div class="nav-rights">
    <div class="nav-right col-xs-12 col-md-10 padding-right-clear">
      <div class="bill_box rightarea padding-right-clear">
        <div class="shaow">
          <div class="money_table">
            <div style="width: 100%;height: 50px;">
              <div style="float:left;letter-spacing:1px;padding-top: 5px;">
                  <span style="font-size:12px;color:#828ea1;">{{$t('uc.Finance.money.totalassets')}}</span>
                  <span style="font-size: 18px;color:#D8E1EB;">${{totalUSDT}}</span>
                  <span style="font-size:10px;color:#828ea1;margin-left: 5px;"> ≈ ¥{{totalCny}}</span>
              </div>
              <Input style="float:right;" class="search" search :placeholder="$t('common.searchplaceholder')" @on-change="seachInputChange" v-model="searchKey"/>

              <Button @click="toQuickExchange" type="warning" style="padding: 6px 30px;margin-right:30px;background-color:#f0a70a;border-color:#f0a70a;float:right;">{{$t('uc.Finance.record.quickExchange')}}</Button>
            </div>
            <Table :no-data-text="$t('common.nodata')" :columns="tableColumnsMoney" :data="tableMoneyShow" :loading="loading" :disabled-hover="true"></Table>
          </div>
        </div>
      </div>
    </div>
    <Modal v-model="modal" :title="$t('uc.Finance.money.match')" @on-ok="matchGCC">
      <P style="font-weight: bold;padding: 10px 0;">{{$t('uc.Finance.money.matchtip1')}}：{{GCCMatchAmount}}</p>
      <p>
        <span>{{$t('uc.Finance.money.matchtip2')}}：</span>
        <InputNumber style="width: 150px;" type="text" v-model="matchAmount" :placeholder="$t('uc.Finance.money.matchtip2')"></InputNumber>
      </p>
    </Modal>
    <Modal v-model="modal_msg" :title="$t('uc.Finance.money.match')">
      <p>{{match_msg}}</p>
    </Modal>
    <Modal v-model="transModal"  @on-ok="confrimTrans" :title="$t('uc.Finance.money.onkeytrans')">
      <p>{{$t('uc.Finance.record.chooseTransCoinUnit')}}：
        <Select v-model="toUnit" style="width: 400px;" :placeholder="$t('common.pleaseselect')">
          <Option v-for="item in transList" :value="item" :key="item">{{ item }}</Option>
        </Select>
      </p>
      <p style="margin-top: 15px;">{{$t('uc.Finance.record.inputTransAmount')}}：
        <InputNumber style="width: 400px;" type="text" v-model="transAmount" :placeholder="$t('uc.Finance.money.matchtip2')"></InputNumber>
      </p>
    </Modal>


    <Modal v-model="quickExchangeModal"  @on-ok="confrimExchange" :title="$t('uc.Finance.record.quickExchange')">
      <h2 style="text-align:center;">{{$t('uc.Finance.record.currentRate')}}： <span style="color: #45b854">{{priceRate | fixed2}} </span></h2>
      <p style="margin-top: 15px;">{{$t('uc.Finance.record.from')}}：
        <Select v-model="fromExchangeCoin" style="width: 450px;" :placeholder="$t('common.pleaseselect')">
          <Option v-for="item in fromCoinList" :value="item" :key="item">{{ item }}</Option>
        </Select>
      </p>
      <p style="margin-top: 15px;">{{$t('uc.Finance.record.to')}}：
        <Select v-model="toExchangeCoin" style="width: 450px;" :placeholder="$t('common.pleaseselect')">
          <Option v-for="item in toCoinList" :value="item" :key="item">{{ item }}</Option>
        </Select>
      </p>
      <p style="margin-top: 15px;">{{$t('uc.Finance.record.inputexchangeamount')}}:
        <InputNumber style="width: 330px;" type="text" v-model="exchangeAmount" :placeholder="$t('uc.Finance.record.inputexchangeamount')"></InputNumber>
        <span style="margin-left: 10px;font-weight: bold;">{{fromExchangeCoin}}</span>
      </p>
      <p style="margin-top: 15px;">{{$t('uc.Finance.record.predictAmount')}}:
        <InputNumber style="width: 330px;" disabled type="text" v-model="predictAmount"></InputNumber>
        <span style="margin-left: 10px;font-weight: bold;">{{toExchangeCoin}}</span>
      </p>
      <p style="margin-top: 15px;">{{$t('uc.Finance.record.inputexchangepasswd')}}:
        <Input style="width: 400px;" type="password" v-model="exchangePassword" :placeholder="$t('uc.Finance.record.inputexchangepasswd')"></Input>
      </p>
    </Modal>
  </div>
</template>
<script>
export default {
  components: {},
  data() {
    return {
      GCCMatchAmount: 0,
      matchAmount: 0,
      modal: false,
      loginmsg: this.$t("common.logintip"),
      loading: true,
      ordKeyword: "",
      tableMoney: [],
      tableMoneyShow: [],
      canMatch: true,
      modal_msg: false,
      match_msg: "",
      searchKey: "",
      transModal: false,
      toUnit: "",
      fromUnit: "",
      transAmount: 0,
      transList: ["USDT", "EUSDT", "TUSDT"],
      quickExchangeModal: false,
      fromExchangeCoin: "CCASH",
      toExchangeCoin: "USDT",
      fromCoinList: ["CCASH"],
      toCoinList: ["USDT"],
      exchangeAmount: 0,
      exchangePassword: "",
      predictAmount: 0,
      priceRate: 7.0
    };
  },
  filters: {
    fixed2: function(value) {
      return value.toFixed(2);
    }
  },
  methods: {
    seachInputChange(){
      this.tableMoneyShow = this.tableMoney.filter(item => item["coinType"].indexOf(this.searchKey) == 0);
    },
    toQuickExchange(){
      this.quickExchangeModal = true;
    },
    confrimExchange(){
      let params = {};
      params["toUnit"] = this.toExchangeCoin;
      params["fromUnit"] = this.fromExchangeCoin;
      params["amount"] = this.transAmount;
      params["jyPassword"] = this.exchangePassword;
      this.$http
        .post(this.host + "/uc/asset/wallet/quick-exchange", params)
        .then(response => {
          var resp = response.body;
          if (resp.code == 0) {
            this.$Message.success(resp.message);
          } else {
            this.$Message.error(resp.message);
          }
          this.quickExchangeModal = false;
        });
      return false;
    },
    getMoney() {
      //获取
      this.$http.post(this.host + "/uc/asset/wallet").then(response => {
        var resp = response.body;
        if (resp.code == 0) {
          this.tableMoney = resp.data;
          for (let i = 0; i < this.tableMoney.length; i++) {
            this.tableMoney[i]["coinType"] = this.tableMoney[i].coin.unit;
          }
          this.loading = false;
          this.tableMoneyShow = this.tableMoney;
        } else {
          this.$Message.error(this.loginmsg);
        }
      });
    },
    getGCCMatchAmount() {
      //获取
      this.$http
        .post(this.host + "/uc/asset/wallet/match-check")
        .then(response => {
          var resp = response.body;
          if (resp.code == 0) {
            this.canMatch = true;
            this.GCCMatchAmount = resp.data;
          } else {
            this.canMatch = false;
            this.match_msg = resp.message;
            // this.$Message.error(this.loginmsg);
          }
          this.showMatchDailog();
        });
    },
    getRate(){
      this.$http
        .post(this.host + "/market/ctc-usdt")
        .then(response => {
          var resp = response.body;
          this.priceRate = resp.data.buy;
        });
    },
    showMatchDailog() {
      if (this.canMatch) this.modal = true;
      else this.modal_msg = true;
    },
    matchGCC() {
      if (this.matchAmount <= 0) {
        this.$Message.warning(this.$t("uc.Finance.money.matcherr1"));
      } else if (this.matchAmount > this.GCCMatchAmount) {
        this.$Message.warning(this.$t("uc.Finance.money.matcherr2"));
      } else {
        //配对
        let params = {};
        params["amount"] = this.matchAmount;
        this.$http
          .post(this.host + "/uc/asset/wallet/match", params)
          .then(response => {
            var resp = response.body;
            if (resp.code == 0) {
              this.$Message.success(this.$t("uc.Finance.money.matchsuccess"));
              this.GCCMatchAmount = this.GCCMatchAmount - this.matchAmount;
            } else {
              this.$Message.error(resp.message);
            }
          });
      }
    },
    resetAddress(unit) {
      this.$Spin.show({
          render: (h) => {
              return h('div', [
                  h('Icon', {
                      'class': 'demo-spin-icon-load',
                      props: {
                          type: 'ios-loading',
                          size: 18
                      }
                  }),
                  h('div', {style:{
                      fontSize: "12px",
                      marginTop: "8px"
                    }}, this.$t('uc.Finance.recharge.gettingaddress'))
              ])
          }
      });
      var self = this;
      let params = {};
      params["unit"] = unit;
      this.$http
        .post(this.host + "/uc/asset/wallet/reset-address", params)
        .then(response => {
          var resp = response.body;
          if (resp.code == 0) {
            //this.$Message.success(this.$t("uc.Finance.money.resetsuccess"));

            setTimeout(function() {
              self.$Spin.hide();
              self.$router.push(
                "/uc/recharge?name=" + unit
              );
            }, 3000);
          } else {
            this.$Message.error(resp.message);
            this.$Spin.hide();
          }
        });
    },
    confrimTrans: function(){
      let params = {};
      params["toUnit"] = this.toUnit;
      params["fromUnit"] = this.fromUnit;
      params["amount"] = this.transAmount;
      this.$http
        .post(this.host + "/uc/asset/wallet/trans-usd", params)
        .then(response => {
          var resp = response.body;
          if (resp.code == 0) {
            this.$Message.success(resp.message);
          } else {
            this.$Message.error(resp.message);
          }
          this.transModal = false;
        });
      return false;
    }
  },
  created() {
    this.getMoney();
  },
  computed: {
    totalUSDT() {
      let usdtTotal = 0;
      for (let i = 0; i < this.tableMoney.length; i++) {
        usdtTotal +=  (this.tableMoney[i].balance + this.tableMoney[i].frozenBalance) * this.tableMoney[i].coin.usdRate;
      }
      return usdtTotal.toFixed(2);
    },
    totalCny(){
      let cnyTotal = 0;
      for (let i = 0; i < this.tableMoney.length; i++) {
        cnyTotal +=  (this.tableMoney[i].balance + this.tableMoney[i].frozenBalance) * this.tableMoney[i].coin.cnyRate;
      }
      return cnyTotal.toFixed(2);
    },
    tableColumnsMoney() {
      let self = this;
      let columns = [];
      columns.push({
        title: this.$t("uc.Finance.money.cointype"),
        key: "coinType",
        width: 100,
        align: "center"
      });
      columns.push({
        title: this.$t("uc.Finance.money.balance"),
        key: "balance",
        align: "center",
        render(h, params) {
          return h(
            "span",
            {
              attrs: {
                title: params.row.balance
              }
            },
            self.toFloor(params.row.balance || "0")
          );
        }
      });
      columns.push({
        title: this.$t("uc.Finance.money.frozen"),
        key: "frozenBalance",
        align: "center",
        render(h, params) {
          return h(
            "span",
            {
              attrs: {
                title: params.row.frozenBalance
              }
            },
            self.toFloor(params.row.frozenBalance || "0")
          );
        }
      });
      columns.push({
        title: this.$t("uc.Finance.money.needreleased"),
        align: "center",
        render(h, params) {
          return h(
            "span",
            {
              attrs: {
                title: params.row.toReleased
              }
            },
            self.toFloor(params.row.toReleased || "0")
          );
        }
      });
      columns.push({
        title: this.$t("uc.Finance.money.operate"),
        key: "price1",
        align: "center",
        render: function(h, params) {
          var actions = [];
          if (params.row.coin.canRecharge == 1) {
            if ( (params.row.address != null && params.row.address != "") || (params.row.coin.accountType == 1)) {
              // 充币
              actions.push(
                h(
                  "Button",
                  {
                    // 充币;
                    props: {
                      type: "info",
                      size: "small"
                    },
                    on: {
                      click: function() {
                        self.$router.push(
                          "/uc/recharge?name=" + params.row.coin.unit
                        );
                      }
                    },
                    style: {
                      marginRight: "8px"
                    }
                  },
                  self.$t("uc.Finance.money.charge")
                )
              );
            } else {
              //   获取地址按钮;
              actions.push(
                h(
                  "Button",
                  {
                    props: {
                      type: "info",
                      size: "small"
                    },
                    on: {
                      click: function() {
                        self.resetAddress(params.row.coin.unit);
                      }
                    },
                    style: {
                      marginRight: "8px"
                    }
                  },
                  self.$t("uc.Finance.money.charge")
                )
              );
            }
          }else{
            actions.push(
              h(
                "Button",
                {
                  props: {
                    size: "small",
                    disabled: true
                  },
                  on: {
                    click: function() {

                    }
                  },
                  style: {
                    marginRight: "8px"
                  }
                },
                self.$t("uc.Finance.money.charge")
              )
            );
          }
          if (params.row.coin.canWithdraw == 1) {
            // 提币;
            actions.push(
              h(
                "Button",
                {
                  props: {
                    type: "error",
                    size: "small"
                  },
                  on: {
                    click: function() {
                      self.$router.push(
                        "/uc/withdraw?name=" + params.row.coin.unit
                      );
                    }
                  },
                  style: {
                    marginRight: "8px"
                  }
                },
                self.$t("uc.Finance.money.pickup")
              )
            );
          }else{
            actions.push(
              h(
                "Button",
                {
                  props: {
                    size: "small",
                    disabled: true
                  },
                  on: {
                    click: function() {

                    }
                  },
                  style: {
                    marginRight: "8px"
                  }
                },
                self.$t("uc.Finance.money.pickup")
              )
            );
          }
          if(params.row.coin.unit == "USDT" || params.row.coin.unit == "EUSDT" || params.row.coin.unit == "TUSDT") {
            actions.push(
              h(
                "Button",
                {
                  props: {
                    type: "success",
                    size: "small",
                    disabled: false
                  },
                  on: {
                    click: function() {
                      self.fromUnit = params.row.coin.unit;
                      self.transModal = true;
                    }
                  },
                  style: {
                    marginRight: "8px"
                  }
                },
                self.$t("uc.Finance.money.onkeytrans")
              )
            );
          }
          return h("p", actions);
        }
      });
      return columns;
    }
  }
};
</script>
<style lang="scss">
.nav-right {
  .rightarea.bill_box {
    .shaow {
      padding: 5px;
    }
    .money_table {
      .search{
        width: 200px;
        margin-bottom: 10px;
      }
      .ivu-table-wrapper {
        .ivu-table-header{
          background: #27313e;
          th{
            color: #fff;
          }
        }
        .ivu-table-body {
          td {
            color: #fff;
            .ivu-table-cell {
              padding: 10px 10px;
              p .ivu-btn {
                background: transparent;
                height: 25px;
                padding: 0 0px;
                border-radius: 0;
                span {
                  display: inline-block;
                  line-height: 20px;
                  font-size: 12px;
                  padding: 0 15px;
                  letter-spacing: 1px;
                }
              }
              p .ivu-btn.ivu-btn-info {
                border: 1px solid #f0ac19;
                span {
                  color: #f0ac19;
                }
              }
              p .ivu-btn.ivu-btn-error {
                border: 1px solid #f15057;
                span {
                  color: #f15057;

                }
              }
              p .ivu-btn.ivu-btn-primary {
                border: 1px solid #00b275;
                border: 1px solid #00b275;
                span {
                  color: #00b275;
                }
              }
              p .ivu-btn.ivu-btn-default {
                border: 1px solid #2c384f;
                background: #222c3e;
                span {
                  color: #54637a;
                }
              }
              p .ivu-btn.ivu-btn-success {
                border: 1px solid #32ad00;
                span {
                  color: #32ad00;
                }
              }
            }
          }
        }
      }
    }
  }
}

.ivu-input-number-disabled .ivu-input-number-input{
  background: transparent!important;
}
</style>

<style scoped lang="scss">
.nav-right {
  height: auto;
  overflow: hidden;
  padding: 0 0 0 15px;
  .rightarea.bill_box {
    padding-left: 15px;
    width: 100%;
    height: auto;
    overflow: hidden;
  }
}

.demo-spin-icon-load{
  animation: ani-demo-spin 1s linear infinite;
}
@media screen and (max-width:768px){
  .search{
    display: none;
  }
}

</style>
