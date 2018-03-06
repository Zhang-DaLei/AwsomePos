<template>
  <div class="pos">
    <el-row>
      <el-col :span='7' class="pos-order" id="order-list">
        <el-tabs>
          <el-tab-pane label="点餐">
            <el-table :data="tableData" border style="width:100%">
              <el-table-column prop="goodsName" label="商品名称"></el-table-column>
              <el-table-column prop="count" label="数量" width="50"></el-table-column>
              <el-table-column prop="price" label="金额" width="70"></el-table-column>
              <el-table-column label="操作" fixed="right" width="100">
                <template slot-scope="scope">
                  <el-button type="text" size="small" @click="delSingleList(scope.row)">删除 </el-button>
                  <el-button type="text" size="small" @click="addList(scope.row)">添加 </el-button>
                </template>
              </el-table-column>
            </el-table>
            <div class="sum">
              <small>金额：{{totalMoney}}</small>  <small>数量：{{totalCount}}</small>
            </div>
            <div class="div-btn">
              <el-button type="warning">挂单</el-button>
              <el-button type="danger" @click="delAllGoods">删除</el-button>
              <el-button type="success" @click="checkout">结账</el-button>
            </div>
          </el-tab-pane>
          <el-tab-pane label="挂单">挂单</el-tab-pane>
          <el-tab-pane label="外卖">外卖</el-tab-pane>
        </el-tabs>
      </el-col>
      <el-col :span="17" class="right-sel">
        <div class="often-goods">
          <div class="title">常用商品</div>
          <div class="often-goods-list">
            <ul>
              <li v-for="item in oftenGoods" :key="item.id" @click="addList(item)">
                <span>{{item.goodsName}}</span>
                <span class="o-price">¥{{item.price}}元</span>
              </li>
            </ul>
          </div>
        </div>
        <div class="goods-type">

          <el-tabs>
            <el-tab-pane label="汉堡">
              <div>
                <ul class='cookList'>
                  <li v-for="item in type0Goods" :key="item.goodsId" @click="addList(item)">
                    <span class="foodImg">
                      <img :src=item.goodsImg width="100%">
                    </span>
                    <span class="foodName">{{item.goodsName}}</span>
                    <span class="foodPrice">¥{{item.price}}元</span>
                  </li>
                </ul>
              </div>
            </el-tab-pane>
            <el-tab-pane label="小食">
              <div>
                <ul class='cookList'>
                  <li v-for="item in type1Goods" :key="item.goodsId" @click="addList(item)">
                    <span class="foodImg">
                      <img :src=item.goodsImg width="100%">
                    </span>
                    <span class="foodName">{{item.goodsName}}</span>
                    <span class="foodPrice">¥{{item.price}}元</span>
                  </li>
                </ul>
              </div>
            </el-tab-pane>
            <el-tab-pane label="饮料">
              <div>
                <ul class='cookList'>
                  <li v-for="item in type2Goods" :key="item.goodsId" @click="addList(item)">
                    <span class="foodImg">
                      <img :src=item.goodsImg width="100%">
                    </span>
                    <span class="foodName">{{item.goodsName}}</span>
                    <span class="foodPrice">¥{{item.price}}元</span>
                  </li>
                </ul>
              </div>
            </el-tab-pane>
            <el-tab-pane label="套餐">
              <div>
                <ul class='cookList'>
                  <li v-for="item in type3Goods" :key="item.goodsId">
                    <span class="foodImg">
                      <img :src=item.goodsImg width="100%">
                    </span>
                    <span class="foodName">{{item.goodsName}}</span>
                    <span class="foodPrice">¥{{item.price}}元</span>
                  </li>
                </ul>
              </div>
            </el-tab-pane>

          </el-tabs>
        </div>
      </el-col>
    </el-row>
  </div>
</template>
<script>
  import axios from 'axios'
  export default {
    name: "pos",
    data() {
      return {
        tableData: [],
        oftenGoods:[],
        type0Goods:[],
        type1Goods:[],
        type2Goods:[],
        type3Goods:[],
        totalCount:0,
        totalMoney:0
      }
    },
    created(){
      axios.get('http://jspang.com/DemoApi/oftenGoods.php')
      .then(response=>{
        this.oftenGoods=response.data;
      }).catch(error=>{
        console.log(error);
        alert('网络错误，不能访问');
      });

      axios.get('http://jspang.com/DemoApi/typeGoods.php')
      .then(response=>{
        this.type0Goods=response.data[0];
        this.type1Goods=response.data[1];
        this.type2Goods=response.data[2];
        this.type3Goods=response.data[3];
      }).catch(error=>{
        console.log(error);
        alert('网络错误，不能访问');
      })
    },
    mounted: function () {
      var orderHeight = document.body.clientHeight;
      document.getElementById('order-list').style.height = orderHeight + 'px'
    },
    methods:{
      addList(goods){
            this.totalCount=0; //汇总数量清0
            this.totalMoney=0;
            let isHave=false;
            //判断是否这个商品已经存在于订单列表
            for (let i=0; i<this.tableData.length;i++){
             //   console.log(this.tableData[i].goodsId);
                if(this.tableData[i].goodsId==goods.goodsId){
                    isHave=true; //存在
                }
            }
            //根据isHave的值判断订单列表中是否已经有此商品
            if(isHave){
                //存在就进行数量添加
                 let arr = this.tableData.filter(function(o){
                    return o.goodsId == goods.goodsId//这里只返回与当前点击的商品有关的信信息
                 } );          
                 arr[0].count++;
                 
            }else{
                //不存在就推入数组
                let newGoods={goodsId:goods.goodsId,goodsName:goods.goodsName,price:goods.price,count:1};
                 this.tableData.push(newGoods); 
            }

            this.computedSum();        
      },
      delSingleList(goods){
        this.tableData=this.tableData.filter(function(o){
          return o.goodsId!=goods.goodsId;
        })
        this.computedSum();
      },
      computedSum(){
        this.totalCount=0;
        this.totalMoney=0;
        if(this.tableData){
            //进行数量和价格的汇总计算
            this.tableData.forEach((element) => {
                this.totalCount+=element.count;
                this.totalMoney=this.totalMoney+(element.price*element.count);   
            });
        }
      },
      delAllGoods(){
        this.tableData=[];
        this.totalMoney=0;
        this.totalCount=0;
      },
      checkout() {
          if (this.totalCount!=0) {
              this.tableData = [];
              this.totalCount = 0;
              this.totalMoney = 0;
              this.$message({
                  message: '结账成功，感谢你又为店里出了一份力!',
                  type: 'success'
              });
      
          }else{
              this.$message.error('不能空结。老板了解你急切的心情！');
          }
      
      }
    }
  }

  ;

</script>
< !-- Add "scoped" attribute to limit CSS to this component only -->
  <style scoped>
    .pos-order {
      background-color: #F9FAFC;
      border-right: 1px solid #C0CCDA;
    }

    .div-btn {
      margin-top: 20px;
    }

    .title {
      height: 18px;
      border-bottom: 1px solid #D3DCE6;
      background-color: #F9FAFC;
      padding: 10px;
    }

    .often-goods-list ul{
      display: flex;
      flex-direction: row;
      flex-wrap: wrap;
    }
    .often-goods-list ul li {
      list-style: none;
      border: 1px solid #E5E9F2;
      padding: 10px;
      margin: 5px;
      background-color: #fff;
      width: 150px;
      cursor: pointer;
    }

    .o-price {
      color: #58B7FF;
    }

    .right-sel {
      display: flex;
      flex-direction: column;
    }

    .goods-type {
      margin-top: 20px;
    }
    .cookList{
      display: flex;
      flex-direction: row;
      flex-wrap: wrap;
    }
    .cookList li{
       list-style: none;
       width:23%;
       border:1px solid #E5E9F2;
       height: auto;
       overflow: hidden;
       background-color:#fff;
       padding: 2px;
       margin: 2px;
      cursor: pointer;
   }
   .cookList li span{
       
        display: block;
        float:left;
   }
   .foodImg{
       width: 40%;
   }
   .foodName{
       font-size: 18px;
       padding-left: 10px;
       color:brown;
 
   }
   .foodPrice{
       font-size: 16px;
       padding-left: 10px;
       padding-top:10px;
   }
   .sum{
     display: flex;
   }
   .sum small{
     width: 50%;
     font-size: 14px;
     padding-top: 4px;
   }
  </style>
