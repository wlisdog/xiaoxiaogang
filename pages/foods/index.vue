<template>
  <view class="container">
    <view class="main" v-if="goods.length">
      <view class="nav">
        <view class="header">
          <view class="left" v-if="orderType == 'takein'">
            <view class="store-name">
              <text>{{ store.name }}</text>
              <view class="iconfont iconarrow-right"></view>
            </view>
            <view class="store-location">
              <image
                src="/static/images/order/location.png"
                style="width: 30rpx; height: 30rpx"
                class="mr-10"
              ></image>
              <text>距离您 {{ store.distance_text }}</text>
            </view>
          </view>

          <view class="right">
            <view class="dinein" :class="{ active: orderType == 'takein' }">
              <text>自取</text>
            </view>
            <view class="takeout" :class="{ active: orderType == 'takeout' }">
              <text>外卖</text>
            </view>
          </view>
        </view>
        <view class="coupon">
          <text class="title">"霸气mini卡"超级购券活动，赶紧去购买</text>
          <view class="iconfont iconarrow-right"></view>
        </view>
      </view>
      <view class="content">
        <scroll-view
          class="menus"
          :scroll-into-view="menuScrollIntoView"
          scroll-with-animation
          scroll-y
        >
          <view class="wrapper">
            <view
              class="menu"
              :id="`menu-${item.id}`"
              :class="{ current: item.id === currentCateId }"
              v-for="(item, index) in goods"
              :key="index"
              @tap="handleMenuTap(item.id)"
            >
              <text>{{ item.name }}</text>
              <view class="dot" v-show="menuCartNum(item.id)">{{
                menuCartNum(item.id)
              }}</view>
            </view>
          </view>
        </scroll-view>
        <!-- goods list begin -->
        <scroll-view
          class="goods"
          scroll-with-animation
          scroll-y
          :scroll-top="cateScrollTop"
          @scroll="handleGoodsScroll"
        >
          <view class="wrapper">
            <swiper
              class="ads"
              id="ads"
              autoplay
              :interval="3000"
              indicator-dots
            >
              <swiper-item v-for="(item, index) in ads" :key="index">
                <image :src="item.image"></image>
              </swiper-item>
            </swiper>
            <view class="list">
              <!-- category begin -->
              <view
                class="category"
                v-for="(item, index) in goods"
                :key="index"
                :id="`cate-${item.id}`"
              >
                <view class="title">
                  <text>{{ item.name }}</text>
                  <image :src="item.icon" class="icon"></image>
                </view>
                <view class="items">
                  <!-- 商品 begin -->
                  <view
                    class="good"
                    v-for="(good, key) in item.goods_list"
                    :key="key"
                  >
                    <image
                      :src="good.images"
                      class="image"
                      @tap="showGoodDetailModal(item, good)"
                    ></image>
                    <view class="right">
                      <text class="name">{{ good.name }}</text>
                      <text class="tips">{{ good.content }}</text>
                      <view class="price_and_action">
                        <text class="price">￥{{ good.price }}</text>
                        <view class="btn-group" v-if="good.use_property">
                          <button
                            type="primary"
                            class="btn property_btn"
                            hover-class="none"
                            size="mini"
                            @tap="showGoodDetailModal(item, good)"
                          >
                            选规格
                          </button>
                          <view class="dot" v-show="goodCartNum(good.id)">{{
                            goodCartNum(good.id)
                          }}</view>
                        </view>
                        <view class="btn-group" v-else>
                          <button
                            type="default"
                            v-show="goodCartNum(good.id)"
                            plain
                            class="btn reduce_btn"
                            size="mini"
                            hover-class="none"
                            @tap="handleReduceFromCart(item, good)"
                          >
                            <view class="iconfont iconsami-select"></view>
                          </button>
                          <view class="number" v-show="goodCartNum(good.id)">{{
                            goodCartNum(good.id)
                          }}</view>
                          <button
                            type="primary"
                            class="btn add_btn"
                            size="min"
                            hover-class="none"
                            @tap="handleAddToCart(item, good, 1)"
                          >
                            <view class="iconfont iconadd-select"></view>
                          </button>
                        </view>
                      </view>
                    </view>
                  </view>
                  <!-- 商品 end -->
                </view>
              </view>
              <!-- category end -->
            </view>
          </view>
        </scroll-view>
        <!-- goods list end -->
      </view>
      <!-- content end -->
      <!-- 购物车栏 begin -->
      <view class="cart-box" v-if="cart.length > 0">
        <view class="mark">
          <image
            src="/static/images/menu/cart.png"
            class="cart-img"
            @tap="openCartPopup"
          ></image>
          <view class="tag">{{ getCartGoodsNumber }}</view>
        </view>
        <view class="price">￥{{ getCartGoodsPrice }}</view>
        <button
          type="primary"
          class="pay-btn"
          @tap="toPay"
          :disabled="disabledPay"
        >
          {{ disabledPay ? `差${spread}元起送` : "去结算" }}
        </button>
      </view>
      <!-- 购物车栏 end -->
    </view>
    <!-- 商品详情模态框 begin -->
    <modal :show="goodDetailModalVisible" class="good-detail-modal" color="#5A5B5C" 
				width="90%" custom padding="0rpx" radius="12rpx">
			<view class="cover">
				<image v-if="good.images" :src="good.images" class="image"></image>
				<view class="btn-group">
					<image src="/static/images/menu/share-good.png"></image>
					<image src="/static/images/menu/close.png" @tap="closeGoodDetailModal"></image>
				</view>
			</view>
			<scroll-view class="detail" scroll-y>
				<view class="wrapper">
					<view class="basic">
						<view class="name">{{ good.name }}</view>
						<view class="tips">{{ good.content }}</view>
					</view>
					<view class="properties" v-if="good.use_property">
						<view class="property" v-for="(item, index) in good.property" :key="index">
							<view class="title">
								<text class="name">{{ item.name }}</text>
								<view class="desc" v-if="item.desc">({{ item.desc }})</view>
							</view>
							<view class="values">
								<view class="value" v-for="(value, key) in item.values" :key="key" 
								:class="{'default': value.is_default}" 
								@tap="changePropertyDefault(index, key)">
									{{ value.value }}
								</view>
							</view>
						</view>
					</view>
				</view>
			</scroll-view>
			<view class="action">
				<view class="left">
					<view class="price">￥{{ good.price }}</view>
					<view class="props" v-if="getGoodSelectedProps(good)">
						{{ getGoodSelectedProps(good) }}
					</view>
				</view>
				<view class="btn-group">
					<button type="default" plain class="btn" size="mini" hover-class="none" 
						@tap="handlePropertyReduce">
						<view class="iconfont iconsami-select"></view>
					</button>
					<view class="number">{{ good.number }}</view>
					<button type="primary" class="btn" size="min" hover-class="none" 
						@tap="handlePropertyAdd">
						<view class="iconfont iconadd-select"></view>
					</button>
				</view>
			</view>
			<view class="add-to-cart-btn" @tap="handleAddToCartInModal">
				<view>加入购物车</view>
			</view>
		</modal>
    <!-- 商品详情模态框 end -->
    <!-- 购物车详情popup -->
    <popup-layer
      direction="top"
      :show-pop="cartPopupVisible"
      class="cart-popup"
    >
      <template slot="content">
        <view class="top">
          <text @tap="handleCartClear">清空</text>
        </view>
        <scroll-view class="cart-list" scroll-y>
          <view class="wrapper">
            <view class="item" v-for="(item, index) in cart" :key="index">
              <view class="left">
                <view class="name">{{ item.name }}</view>
                <view class="props">{{ item.props_text }}</view>
              </view>
              <view class="center">
                <text>￥{{ item.price }}</text>
              </view>
              <view class="right">
                <button
                  type="default"
                  plain
                  size="mini"
                  class="btn"
                  hover-class="none"
                  @tap="handleCartItemReduce(index)"
                >
                  <view class="iconfont iconsami-select"></view>
                </button>
                <view class="number">{{ item.number }}</view>
                <button
                  type="primary"
                  class="btn"
                  size="min"
                  hover-class="none"
                  @tap="handleCartItemAdd(index)"
                >
                  <view class="iconfont iconadd-select"></view>
                </button>
              </view>
            </view>

            <view
              class="item"
              v-if="orderType == 'takeout' && store.packing_fee"
            >
              <view class="left">
                <view class="name">包装费</view>
              </view>
              <view class="center">
                <text>￥{{ parseFloat(store.packing_fee) }}</text>
              </view>
              <view class="right invisible">
                <button
                  type="default"
                  plain
                  size="mini"
                  class="btn"
                  hover-class="none"
                >
                  <view class="iconfont iconsami-select"></view>
                </button>
                <view class="number">1</view>
                <button
                  type="primary"
                  class="btn"
                  size="min"
                  hover-class="none"
                >
                  <view class="iconfont iconadd-select"></view>
                </button>
              </view>
            </view>
          </view>
        </scroll-view>
      </template>
    </popup-layer>
    <!-- 购物车详情popup -->
  </view>
  <!-- <view class="loading" v-else>
		<image src="/static/images/loading.gif"></image>
	</view> -->
</template>

<script>
import modal from "@/components/modal/modal";
import popupLayer from "@/components/popup-layer/popup-layer";

export default {
  components: {
    modal,
    popupLayer,
  },
  data() {
    return {
      goods: [], //所有商品
      ads: [
        {
          image:
            "https://img-shop.qmimg.cn/s23107/2020/04/27/4ebdb582a5185358c4.jpg?imageView2/2/w/600/h/600",
        },
        {
          image:
            "https://images.qmai.cn/s23107/2020/05/08/c25de6ef72d2890630.png?imageView2/2/w/600/h/600",
        },
        {
          image:
            "https://img-shop.qmimg.cn/s23107/2020/04/10/add546c1b1561f880d.jpg?imageView2/2/w/600/h/600",
        },
        {
          image:
            "https://images.qmai.cn/s23107/2020/04/30/b3af19e0de8ed42f61.jpg?imageView2/2/w/600/h/600",
        },
        {
          image:
            "https://img-shop.qmimg.cn/s23107/2020/04/17/8aeb78516d63864420.jpg?imageView2/2/w/600/h/600",
        },
      ],
      loading: true,
      currentCateId: 6905, //默认分类
      cateScrollTop: 0,
      menuScrollIntoView: "",
      cart: [], //购物车
      goodDetailModalVisible: false, //是否饮品详情模态框
      good: {}, //当前饮品
      category: {}, //当前饮品所在分类
      cartPopupVisible: false,
      sizeCalcState: false,
      orderType: "takein",
      isLogin: true,
      store: {
        longitude: "114.065927",
        latitude: "22.537361",
        area_name: "福田区",
        photo: '["s23107\\\/2019\\\/03\\\/25\\\/e4a12f9f81bd3e8f4d.jpg"]',
        is_show: 1,
        mobile: "0755-82722513",
        takeout_server_status: true,
        is_open: true,
        server_status: true,
        created_at: "1568194697",
        street: "深圳市福田区海田路与福华一路交汇深圳天元5栋1层N136",
        area_id: 440304,
        notice: "",
        city_name: "深圳市",
        id_card: "222222222222222222",
        alipay_store_id: "",
        takeout_server_time: "10:00-23:59",
        id: 1,
        forhere_server_time: "10:00-23:59",
        province_id: 440000,
        forhere_is_open: true,
        is_floor_stall: 0,
        is_eat: 1,
        share_description: "",
        distance: 896,
        distance_text: "896m",
        stalls: [],
        tel: "0755-82722513",
        is_takeout: 1,
        images: [
          "https:\/\/img-shop.qmimg.cn\/s23107\/2019\/03\/25\/e4a12f9f81bd3e8f4d.jpg?imageView2\/0\/w\/200\/h\/200",
        ],
        shop_day: "",
        image:
          "https:\/\/img-shop.qmimg.cn\/s23107\/2019\/03\/25\/e4a12f9f81bd3e8f4d.jpg",
        server_time: "10:00-23:59",
        status: 1,
        multi_mark: "NXHNSZ0055",
        per_price: "1.00",
        balance: "0.00",
        name: "卓悦中心ONE AVENUE店",
        updated_at: "1578227762",
        packing_fee: "2.00",
        delivery_cost: "2.00",
        min_price: "30.00",
        avg_delivery_cost_time: "40",
      },
    };
  },
  async onLoad() {
    await this.init();
  },
  computed: {
    goodCartNum() {
      //计算单个饮品添加到购物车的数量
      return (id) =>
        this.cart.reduce((acc, cur) => {
          if (cur.id === id) {
            return (acc += cur.number);
          }
          return acc;
        }, 0);
    },
    menuCartNum() {
      return (id) =>
        this.cart.reduce((acc, cur) => {
          if (cur.cate_id === id) {
            return (acc += cur.number);
          }
          return acc;
        }, 0);
    },
    getCartGoodsNumber() {
      //计算购物车总数
      return this.cart.reduce((acc, cur) => acc + cur.number, 0);
    },
    getCartGoodsPrice() {
      //计算购物车总价
      return this.cart.reduce((acc, cur) => acc + cur.number * cur.price, 0);
    },
    disabledPay() {
      //是否达到起送价
      return this.orderType == "takeout" &&
        this.getCartGoodsPrice < this.store.min_price
        ? true
        : false;
    },
    spread() {
      //差多少元起送
      if (this.orderType != "takeout") return;
      return parseFloat(
        (this.store.min_price - this.getCartGoodsPrice).toFixed(2)
      );
    },
  },
  methods: {
    async init() {
      //页面初始化
      this.loading = true;
      this.goods = [
        {
          sort: 1,
          icon: "",
          id: 6905,
          goods_list: [
            {
              sell_time_status: 0,
              id: 65825,
              is_sell: true,
              pack_cost: "0.00",
              sales: 487,
              goods_type: 1,
              cover_img: "",
              property: [],
              goods_meals_info: [],
              is_add: 1,
              use_spec: false,
              entity: [
                {
                  spec_id: "",
                  trade_mark: "11110090",
                  id: "9ad36aa96636c246",
                  stock: "9999956.00",
                  spec_text: [],
                  spec: [],
                  image: "s23107/2020/04/19/fda6dd99c83af02353.jpg",
                  num: 1,
                  price: 18.5,
                  membership_price: 0,
                },
              ],
              stall_code: "",
              sort: 99,
              price: 18.5,
              unit: "件",
              imageArr: [
                "https://img-shop.qmimg.cn/s23107/2020/04/19/fda6dd99c83af02353.jpg?imageView2/2/w/600/h/600",
              ],
              membership_price: 0,
              use_property: 0,
              unit_type: 0,
              min_buy_num: 0,
              specs: [],
              content: "购买三明治,享早餐指定饮品半价",
              is_follow_suit: 1,
              stock: "9999956.00",
              type: 2,
              is_label: 0,
              name: "奈雪早餐",
              images:
                "https://img-shop.qmimg.cn/s23107/2020/04/19/fda6dd99c83af02353.jpg?imageView2/2/w/400/h/400",
            },
          ],
          name: "奈雪早餐",
          is_show_backstage: 0,
        },
        {
          sort: 2,
          icon: "https://img-shop.qmimg.cn/s23107/2019/04/30/458c5a14fb2f190f96.png?imageView2/0/w/200/h/200",
          id: 6208,
          goods_list: [
            {
              sell_time_status: 0,
              id: 24516,
              pack_cost: "0.00",
              sales: 1278,
              cover_img: "",
              property: [],
              is_sell: true,
              goods_type: 1,
              entity: [
                {
                  spec_id: "",
                  trade_mark: "1212000070",
                  id: "5d79de67251ea00e",
                  stock: "10485.00",
                  spec_text: [],
                  spec: [],
                  image: "s23107/2020/04/29/4a62ee45dd527609ed.jpg",
                  num: 1,
                  price: 18,
                  membership_price: 0,
                },
              ],
              stall_code: "",
              sort: 3,
              price: 18,
              unit: "件",
              imageArr: [
                "https://img-shop.qmimg.cn/s23107/2020/04/29/4a62ee45dd527609ed.jpg?imageView2/2/w/600/h/600",
              ],
              membership_price: 0,
              use_property: 0,
              unit_type: 0,
              min_buy_num: 0,
              specs: [],
              content: "酥软口感,进口奶油搭配特制巧克力内馅",
              use_spec: false,
              stock: "10485.00",
              type: 1,
              is_label: 0,
              name: "脏脏王",
              images:
                "https://img-shop.qmimg.cn/s23107/2020/04/29/4a62ee45dd527609ed.jpg?imageView2/2/w/400/h/400",
            },
            {
              sell_time_status: 0,
              id: 24517,
              pack_cost: "0.00",
              sales: 1228,
              cover_img: "",
              property: [],
              is_sell: true,
              goods_type: 1,
              entity: [
                {
                  spec_id: "",
                  trade_mark: "1212000071",
                  id: "2b3f1ea3ecabd22e",
                  stock: "10308.00",
                  spec_text: [],
                  spec: [],
                  image: "s23107/2020/04/29/99daa7b20061efab10.jpg",
                  num: 1,
                  price: 18,
                  membership_price: 0,
                },
              ],
              stall_code: "",
              sort: 4,
              price: 18,
              unit: "件",
              imageArr: [
                "https://img-shop.qmimg.cn/s23107/2020/04/29/99daa7b20061efab10.jpg?imageView2/2/w/600/h/600",
              ],
              membership_price: 0,
              use_property: 0,
              unit_type: 0,
              min_buy_num: 0,
              specs: [],
              content: "精选北川半兵卫抹茶粉和秘制红豆泥",
              use_spec: false,
              stock: "10308.00",
              type: 1,
              is_label: 0,
              name: "抹茶王",
              images:
                "https://img-shop.qmimg.cn/s23107/2020/04/29/99daa7b20061efab10.jpg?imageView2/2/w/400/h/400",
            },
            {
              sell_time_status: 0,
              id: 68339,
              pack_cost: "0.00",
              sales: 44429,
              cover_img: "",
              property: [
                {
                  is_open_checkbox: false,
                  id: 190,
                  values: [
                    {
                      is_default: 1,
                      id: 582,
                      code: "Z5EA54C3D0E4279185",
                      value: "标准冰",
                    },
                    {
                      is_default: 0,
                      id: 583,
                      code: "Z5EA54C3D0E9682750",
                      value: "去冰",
                    },
                  ],
                  name: "温度",
                  desc: null,
                },
                {
                  is_open_checkbox: false,
                  id: 191,
                  values: [
                    {
                      is_default: 1,
                      id: 585,
                      code: "Z5EA54C3D0F2A37322",
                      value: "标准糖",
                    },
                    {
                      is_default: 0,
                      id: 586,
                      code: "Z5EA54C3D0F70A1000",
                      value: "少糖",
                    },
                    {
                      is_default: 0,
                      id: 1959,
                      code: "Z5EA54C3D0FAFE7320",
                      value: "不另外加糖",
                    },
                  ],
                  name: "糖度",
                  desc: null,
                },
              ],
              is_sell: true,
              goods_type: 2,
              entity: [
                {
                  spec_id: "",
                  trade_mark: "1010010025",
                  id: "72cf1279c0a422ce",
                  stock: "9999817.00",
                  spec_text: [],
                  spec: [],
                  image: "s23107/2020/04/26/559a075d81060b23c7.jpg",
                  num: 1,
                  price: 30,
                  membership_price: 0,
                },
              ],
              stall_code: "",
              sort: 0,
              price: 30,
              unit: "件",
              imageArr: [
                "https://img-shop.qmimg.cn/s23107/2020/04/26/559a075d81060b23c7.jpg?imageView2/2/w/600/h/600",
              ],
              membership_price: 0,
              use_property: 1,
              unit_type: 0,
              min_buy_num: 0,
              specs: [],
              content:
                "饱满大颗的新鲜杨梅,满杯手工去核,搭配茉莉初雪茶底,清爽多汁",
              use_spec: false,
              stock: "9999817.00",
              type: 1,
              is_label: 0,
              name: "霸气杨梅",
              images:
                "https://img-shop.qmimg.cn/s23107/2020/04/26/559a075d81060b23c7.jpg?imageView2/2/w/400/h/400",
            },
            {
              sell_time_status: 0,
              id: 68345,
              pack_cost: "0.00",
              sales: 16322,
              cover_img: "",
              property: [
                {
                  is_open_checkbox: false,
                  id: 190,
                  values: [
                    {
                      is_default: 1,
                      id: 582,
                      code: "Z5EA55088766CC7824",
                      value: "标准冰",
                    },
                    {
                      is_default: 0,
                      id: 583,
                      code: "Z5EA5508876F271659",
                      value: "去冰",
                    },
                  ],
                  name: "温度",
                  desc: null,
                },
                {
                  is_open_checkbox: false,
                  id: 191,
                  values: [
                    {
                      is_default: 1,
                      id: 585,
                      code: "Z5EA55088776843559",
                      value: "标准糖",
                    },
                    {
                      is_default: 0,
                      id: 586,
                      code: "Z5EA5508877A528469",
                      value: "少糖",
                    },
                    {
                      is_default: 0,
                      id: 1959,
                      code: "Z5EA5508877E5A6908",
                      value: "不另外加糖",
                    },
                  ],
                  name: "糖度",
                  desc: null,
                },
              ],
              is_sell: true,
              goods_type: 2,
              entity: [
                {
                  spec_id: "",
                  trade_mark: "1010010043",
                  id: "c7c6f8fd34040338",
                  stock: "999950.00",
                  spec_text: [],
                  spec: [],
                  image: "s23107/2020/04/26/ea8fc439fddf2f62e3.jpg",
                  num: 1,
                  price: 32,
                  membership_price: 0,
                },
              ],
              stall_code: "",
              sort: 2,
              price: 32,
              unit: "件",
              imageArr: [
                "https://img-shop.qmimg.cn/s23107/2020/04/26/ea8fc439fddf2f62e3.jpg?imageView2/2/w/600/h/600",
              ],
              membership_price: 0,
              use_property: 1,
              unit_type: 0,
              min_buy_num: 0,
              specs: [],
              content: "霸气杨梅和酸奶的首次搭配,甘甜可口",
              use_spec: false,
              stock: "999950.00",
              type: 1,
              is_label: 0,
              name: "霸气酸奶杨梅",
              images:
                "https://img-shop.qmimg.cn/s23107/2020/04/26/ea8fc439fddf2f62e3.jpg?imageView2/2/w/400/h/400",
            },
          ],
          name: "新品推荐",
          is_show_backstage: 0,
        },
        {
          sort: 3,
          icon: "https://img-shop.qmimg.cn/s23107/2019/04/30/458c5a14fb2f190f96.png?imageView2/0/w/200/h/200",
          id: 6387,
          goods_list: [
            {
              sell_time_status: 0,
              id: 31826,
              pack_cost: "0.00",
              sales: 70462,
              cover_img: "",
              property: [],
              is_sell: true,
              goods_type: 1,
              entity: [
                {
                  spec_id: "",
                  trade_mark: "1212000017",
                  id: "2e3f40b8f6decb2a",
                  stock: "1000498.00",
                  spec_text: [],
                  spec: [],
                  image: "s23107/2019/04/28/be484557ff7cfa4dba.jpg",
                  num: 1,
                  price: 28,
                  membership_price: 0,
                },
              ],
              stall_code: "",
              sort: 2,
              price: 28,
              unit: "件",
              imageArr: [
                "https://img-shop.qmimg.cn/s23107/2019/04/28/be484557ff7cfa4dba.jpg?imageView2/2/w/600/h/600",
              ],
              membership_price: 0,
              use_property: 0,
              unit_type: 0,
              min_buy_num: 0,
              specs: [],
              content:
                "马来西亚D24榴莲王果肉+芒果干，爆浆的榴莲馅，这款是榴莲控的最爱。",
              use_spec: false,
              stock: "1000498.00",
              type: 1,
              is_label: 0,
              name: "霸气榴莲王",
              images:
                "https://img-shop.qmimg.cn/s23107/2019/04/28/be484557ff7cfa4dba.jpg?imageView2/2/w/400/h/400",
            },
            {
              sell_time_status: 0,
              id: 31507,
              pack_cost: "0.00",
              sales: 199583,
              cover_img: "",
              property: [
                {
                  is_open_checkbox: false,
                  values: [
                    {
                      is_default: 1,
                      id: 582,
                      code: "Z5E5E2B483FA903805",
                      value: "标准冰",
                    },
                    {
                      is_default: 0,
                      id: 583,
                      code: "Z5E5E2B484016A8529",
                      value: "去冰",
                    },
                    {
                      is_default: 0,
                      id: 584,
                      code: "Z5E5E2B48405642832",
                      value: "热",
                    },
                    {
                      is_default: 0,
                      id: 2544,
                      code: "Z5E5E2B48409616050",
                      value: "温",
                    },
                  ],
                  name: "温度",
                  id: 190,
                },
                {
                  is_open_checkbox: false,
                  id: 191,
                  values: [
                    {
                      is_default: 1,
                      id: 585,
                      code: "Z5DD789B45391F4495",
                      value: "标准糖",
                    },
                    {
                      is_default: 0,
                      id: 586,
                      code: "Z5DD789B453D023182",
                      value: "少糖",
                    },
                    {
                      is_default: 0,
                      id: 1959,
                      code: "Z5DD789B4540531795",
                      value: "不另外加糖",
                    },
                  ],
                  name: "糖度",
                  desc: "茶饮含糖量较低，推荐标准做法，口味更佳",
                },
              ],
              is_sell: true,
              goods_type: 2,
              entity: [
                {
                  spec_id: "",
                  trade_mark: "1010010000",
                  id: "097aad038aeeb0ea",
                  stock: "998101.00",
                  spec_text: [],
                  spec: [],
                  image: "s23107/2019/04/26/2a4b2697bec6f7e502.jpg",
                  num: 1,
                  price: 25,
                  membership_price: 0,
                },
              ],
              stall_code: "",
              sort: 2,
              price: 25,
              unit: "件",
              imageArr: [
                "https://img-shop.qmimg.cn/s23107/2019/04/26/2a4b2697bec6f7e502.jpg?imageView2/2/w/600/h/600",
              ],
              membership_price: 0,
              use_property: 1,
              unit_type: 0,
              min_buy_num: 0,
              specs: [],
              content:
                "奈雪明星产品。优选进口新奇士橙，搭配严选茉莉毛尖茶底，橙意满满。",
              use_spec: false,
              stock: "998101.00",
              type: 1,
              is_label: 0,
              name: "霸气橙子",
              images:
                "https://img-shop.qmimg.cn/s23107/2019/04/26/2a4b2697bec6f7e502.jpg?imageView2/2/w/400/h/400",
            },
            {
              sell_time_status: 0,
              id: 31532,
              pack_cost: "0.00",
              sales: 5104,
              cover_img: "",
              property: [
                {
                  is_open_checkbox: false,
                  values: [
                    {
                      is_default: 1,
                      id: 1218,
                      code: "Z5D612C34C8EF06658",
                      value: "标准(芝士)",
                    },
                    {
                      is_default: 0,
                      id: 1166,
                      code: "Z5D612C34C93705460",
                      value: "芝士换酸奶",
                    },
                  ],
                  name: "配料",
                  id: 347,
                },
                {
                  is_open_checkbox: false,
                  values: [
                    {
                      is_default: 1,
                      id: 582,
                      code: "Z5EA8DBDA4371C5994",
                      value: "标准冰",
                    },
                    {
                      is_default: 0,
                      id: 1164,
                      code: "Z5EA8DBDA43C415239",
                      value: "冰沙",
                    },
                    {
                      is_default: 0,
                      id: 583,
                      code: "Z5EA8DBDA4404E3515",
                      value: "去冰",
                    },
                  ],
                  name: "温度",
                  id: 190,
                },
                {
                  is_open_checkbox: false,
                  id: 191,
                  values: [
                    {
                      is_default: 1,
                      id: 585,
                      code: "Z5DD78286916115856",
                      value: "标准糖",
                    },
                    {
                      is_default: 0,
                      id: 586,
                      code: "Z5DD78286919682277",
                      value: "少糖",
                    },
                    {
                      is_default: 0,
                      id: 1959,
                      code: "Z5DD7828691CB89112",
                      value: "不另外加糖",
                    },
                  ],
                  name: "糖度",
                  desc: "茶饮含糖量较低，推荐标准做法，口味更佳",
                },
              ],
              is_sell: true,
              goods_type: 2,
              entity: [
                {
                  spec_id: "",
                  trade_mark: "1010050008",
                  id: "f8114313a48c5c4a",
                  stock: "1000314.00",
                  spec_text: [],
                  spec: [],
                  image: "s23107/2020/03/19/a3ee1fa72435259a73.jpg",
                  num: 1,
                  price: 28,
                  membership_price: 0,
                },
              ],
              stall_code: "",
              sort: 4,
              price: 28,
              unit: "件",
              imageArr: [
                "https://img-shop.qmimg.cn/s23107/2020/03/19/a3ee1fa72435259a73.jpg?imageView2/2/w/600/h/600",
              ],
              membership_price: 0,
              use_property: 1,
              unit_type: 0,
              min_buy_num: 0,
              specs: [],
              content:
                "精选优质巨峰葡萄、手工去皮去籽、加入带兰桂花香，经中度焙火的金观音茶中，搭配轻盈香甜芝士奶盖",
              use_spec: false,
              stock: "1000314.00",
              type: 1,
              is_label: 0,
              name: "霸气芝士葡萄",
              images:
                "https://img-shop.qmimg.cn/s23107/2020/03/19/a3ee1fa72435259a73.jpg?imageView2/2/w/400/h/400",
            },
            {
              sell_time_status: 0,
              id: 16808,
              pack_cost: "0.00",
              sales: 115495,
              cover_img: "",
              property: [],
              is_sell: true,
              goods_type: 2,
              entity: [
                {
                  spec_id: "",
                  trade_mark: "1212000024",
                  id: "2e89b669b329c292",
                  stock: "10704.00",
                  spec_text: [],
                  spec: [],
                  image: "s23107/2019/04/28/94d8440ab7b4fed802.jpg",
                  num: 1,
                  price: 20,
                  membership_price: 0,
                },
              ],
              stall_code: "",
              sort: 5,
              price: 20,
              unit: "件",
              imageArr: [
                "https://img-shop.qmimg.cn/s23107/2019/04/28/94d8440ab7b4fed802.jpg?imageView2/2/w/600/h/600",
              ],
              membership_price: 0,
              use_property: 0,
              unit_type: 0,
              min_buy_num: 0,
              specs: [],
              content:
                "大大的一根魔法棒，加入淡奶油和进口酵母，配上经典的草莓鲜果。好看的外表和经典的口味造就的一款奈雪明星产品。",
              use_spec: false,
              stock: "10704.00",
              type: 1,
              is_label: 0,
              name: "草莓魔法棒",
              images:
                "https://img-shop.qmimg.cn/s23107/2019/04/28/94d8440ab7b4fed802.jpg?imageView2/2/w/400/h/400",
            },
            {
              sell_time_status: 0,
              id: 31424,
              pack_cost: "0.00",
              sales: 125010,
              cover_img: "",
              property: [
                {
                  is_open_checkbox: false,
                  values: [
                    {
                      is_default: 1,
                      id: 1218,
                      code: "Z5D5BA2BB31A145049",
                      value: "标准(芝士)",
                    },
                    {
                      is_default: 0,
                      id: 1166,
                      code: "Z5D5BA2BB320FA8410",
                      value: "芝士换酸奶",
                    },
                  ],
                  name: "配料",
                  id: 347,
                },
                {
                  is_open_checkbox: false,
                  values: [
                    {
                      is_default: 0,
                      id: 582,
                      code: "Z5E5E2BC316DD78061",
                      value: "标准冰",
                    },
                    {
                      is_default: 1,
                      id: 1164,
                      code: "Z5E5E2BC316A215600",
                      value: "冰沙",
                    },
                    {
                      is_default: 0,
                      id: 583,
                      code: "Z5E5E2BC3171CA8287",
                      value: "去冰",
                    },
                    {
                      is_default: 0,
                      id: 584,
                      code: "Z5E5E2BC317A175989",
                      value: "热",
                    },
                    {
                      is_default: 0,
                      id: 2544,
                      code: "Z5E5E2BC317F896724",
                      value: "温",
                    },
                  ],
                  name: "温度",
                  id: 190,
                },
                {
                  is_open_checkbox: false,
                  id: 191,
                  values: [
                    {
                      is_default: 1,
                      id: 585,
                      code: "Z5DD785286B9689681",
                      value: "标准糖",
                    },
                    {
                      is_default: 0,
                      id: 586,
                      code: "Z5DD785286BCF74540",
                      value: "少糖",
                    },
                    {
                      is_default: 0,
                      id: 1959,
                      code: "Z5DD785286C0687027",
                      value: "不另外加糖",
                    },
                  ],
                  name: "糖度",
                  desc: "茶饮含糖量较低，推荐标准做法，口味更佳",
                },
              ],
              is_sell: true,
              goods_type: 2,
              entity: [
                {
                  spec_id: "",
                  trade_mark: "1010050003",
                  id: "ee375ed5ae7f77eb",
                  stock: "1002053.00",
                  spec_text: [],
                  spec: [],
                  image: "s23107/2019/04/26/1cb88e6cd2fbcefb2a.jpg",
                  num: 1,
                  price: 29,
                  membership_price: 0,
                },
              ],
              stall_code: "",
              sort: 99,
              price: 29,
              unit: "件",
              imageArr: [
                "https://img-shop.qmimg.cn/s23107/2019/04/26/1cb88e6cd2fbcefb2a.jpg?imageView2/2/w/600/h/600",
              ],
              membership_price: 0,
              use_property: 1,
              unit_type: 0,
              min_buy_num: 0,
              specs: [],
              content:
                "奈雪明星产品。选用奈雪自有草莓园新鲜草莓，搭配严选茉莉毛尖茶底，淋上轻盈香滑的芝士奶盖。",
              use_spec: false,
              stock: "1002053.00",
              type: 1,
              is_label: 0,
              name: "霸气芝士草莓",
              images:
                "https://img-shop.qmimg.cn/s23107/2019/04/26/1cb88e6cd2fbcefb2a.jpg?imageView2/2/w/400/h/400",
            },
          ],
          name: "招牌热卖",
          is_show_backstage: 0,
        },
      ];
      this.loading = false;
      this.cart = uni.getStorageSync("cart") || [];
    },
    takout() {
      if (this.orderType == "takeout") return;

      if (!this.isLogin) {
        uni.navigateTo({ url: "/pages/login/login" });
        return;
      }

      uni.navigateTo({
        url: "/pages/address/address?is_choose=true",
      });
    },
    handleMenuTap(id) {
      //点击菜单项事件
      if (!this.sizeCalcState) {
        this.calcSize();
      }

      this.currentCateId = id;
      this.$nextTick(
        () =>
          (this.cateScrollTop = this.goods.find((item) => item.id == id).top)
      );
    },
    handleGoodsScroll({ detail }) {
      //商品列表滚动事件
      if (!this.sizeCalcState) {
        this.calcSize();
      }
      const { scrollTop } = detail;
      let tabs = this.goods.filter((item) => item.top <= scrollTop).reverse();
      if (tabs.length > 0) {
        this.currentCateId = tabs[0].id;
      }
    },
    calcSize() {
      let h = 10;

      let view = uni.createSelectorQuery().select("#ads");
      view
        .fields(
          {
            size: true,
          },
          (data) => {
            h += Math.floor(data.height);
          }
        )
        .exec();

      this.goods.forEach((item) => {
        let view = uni.createSelectorQuery().select(`#cate-${item.id}`);
        view
          .fields(
            {
              size: true,
            },
            (data) => {
              item.top = h;
              h += data.height;
              item.bottom = h;
            }
          )
          .exec();
      });
      this.sizeCalcState = true;
    },
    handleAddToCart(cate, good, num) {
      //添加到购物车
      const index = this.cart.findIndex((item) => {
        if (good.use_property) {
          return item.id === good.id && item.props_text === good.props_text;
        } else {
          return item.id === good.id;
        }
      });
      if (index > -1) {
        this.cart[index].number += num;
      } else {
        this.cart.push({
          id: good.id,
          cate_id: cate.id,
          name: good.name,
          price: good.price,
          number: num,
          image: good.images,
          use_property: good.use_property,
          props_text: good.props_text,
          props: good.props,
        });
      }
    },
    handleReduceFromCart(item, good) {
      const index = this.cart.findIndex((item) => item.id === good.id);
      this.cart[index].number -= 1;
      if (this.cart[index].number <= 0) {
        this.cart.splice(index, 1);
      }
    },
    showGoodDetailModal(item, good) {
      this.good = JSON.parse(JSON.stringify({ ...good, number: 1 }));
      this.category = JSON.parse(JSON.stringify(item));
      this.goodDetailModalVisible = true;
    },
    closeGoodDetailModal() {
      //关闭饮品详情模态框
      this.goodDetailModalVisible = false;
      this.category = {};
      this.good = {};
    },
    changePropertyDefault(index, key) {
      //改变默认属性值
      this.good.property[index].values.forEach((value) =>
        this.$set(value, "is_default", 0)
      );
      this.good.property[index].values[key].is_default = 1;
      this.good.number = 1;
    },
    getGoodSelectedProps(good, type = "text") {
      //计算当前饮品所选属性
      if (good.use_property) {
        let props = [];
        good.property.forEach(({ values }) => {
          values.forEach((value) => {
            if (value.is_default) {
              props.push(type === "text" ? value.value : value.id);
            }
          });
        });
        return type === "text" ? props.join("，") : props;
      }
      return "";
    },
    handlePropertyAdd() {
      this.good.number += 1;
    },
    handlePropertyReduce() {
      if (this.good.number === 1) return;
      this.good.number -= 1;
    },
    handleAddToCartInModal() {
      const product = Object.assign({}, this.good, {
        props_text: this.getGoodSelectedProps(this.good),
        props: this.getGoodSelectedProps(this.good, "id"),
      });
      this.handleAddToCart(this.category, product, this.good.number);
      this.closeGoodDetailModal();
    },
    openCartPopup() {
      //打开/关闭购物车列表popup
      this.cartPopupVisible = !this.cartPopupVisible;
    },
    handleCartClear() {
      //清空购物车
      uni.showModal({
        title: "提示",
        content: "确定清空购物车么",
        success: ({ confirm }) => {
          if (confirm) {
            this.cartPopupVisible = false;
            this.cart = [];
          }
        },
      });
    },
    handleCartItemAdd(index) {
      this.cart[index].number += 1;
    },
    handleCartItemReduce(index) {
      if (this.cart[index].number === 1) {
        this.cart.splice(index, 1);
      } else {
        this.cart[index].number -= 1;
      }
      if (!this.cart.length) {
        this.cartPopupVisible = false;
      }
    },
    toPay() {
      if (!this.isLogin) {
        uni.navigateTo({ url: "/pages/login/login" });
        return;
      }

      uni.showLoading({ title: "加载中" });
      uni.setStorageSync("cart", JSON.parse(JSON.stringify(this.cart)));

      uni.navigateTo({
        url: "/pages/pay/pay",
      });
      uni.hideLoading();
    },
  },
};
</script>

<style lang="scss" scoped>
@import "~@/pages/foods/index.scss";
</style>
