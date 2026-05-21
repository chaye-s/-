<template>
  <div class="container">
    <!-- 🔹首次进入弹窗（填写个人信息） -->
    <div v-if="showUserPopup" class="user-popup">
      <div class="popup-box">
        <h3>欢迎使用校园快递代取</h3>
        <p class="tip">填写信息，下单更快捷，后续可在设置修改</p>
        <input v-model="userInfo.nickname" placeholder="昵称">
        <input v-model="userInfo.building" placeholder="宿舍楼栋（例：1栋）">
        <input v-model="userInfo.room" placeholder="宿舍门牌号（例：402）">
        <button class="btn-primary" @click="saveUserInfo">保存进入</button>
      </div>
    </div>

    <!-- 顶部导航 -->
    <header class="header">
      <div class="logo">校园快递代取</div>
      <nav class="nav">
        <a href="#home">首页</a>
        <a href="#service">服务</a>
        <a href="#price">价格</a>
        <a href="#order">下单</a>
        <a href="#myorder">我的订单</a>
        <a href="#comment">用户评价</a>
        <a href="#setting">个人设置</a>
        <a href="#admin" @click="isAdmin = !isAdmin">订单后台(仅你可见)</a>
      </nav>
    </header>

    <!-- 🔹个人设置页面 -->
    <section id="setting" class="section setting">
      <h2 class="title">⚙️ 个人信息设置</h2>
      <div class="setting-box">
        <input v-model="userInfo.nickname" placeholder="昵称">
        <input v-model="userInfo.building" placeholder="宿舍楼栋（例：1栋）">
        <input v-model="userInfo.room" placeholder="宿舍门牌号（例：402）">
        <button class="btn-primary" @click="saveUserInfo">保存信息</button>
      </div>
    </section>

    <!-- 🔹我的订单（顾客查看自己订单+状态） -->
    <section id="myorder" class="section myorder">
      <h2 class="title">📦 我的订单</h2>
      <div class="myorder-box">
        <div v-if="myOrderList.length === 0" class="empty">暂无订单，快去下单吧</div>
        <div v-for="(item, index) in myOrderList" :key="index" class="myorder-item">
          <p><strong>服务：</strong>{{item.typeName}}｜{{item.deliverName}}</p>
          <p><strong>预估价格：</strong>{{item.price}} 元</p>
          <p><strong>实际价格：</strong>{{item.realPrice}} 元</p>
          <p><strong>需补差价：</strong>{{Math.max(Number(item.realPrice)-Number(item.price), 0)}} 元</p>
          <p><strong>状态：</strong>
            <span class="status-tag" :class="item.status">
              {{item.status === 'wait' ? '待取件' : item.status === 'send' ? '配送中' : '已送达'}}
            </span>
          </p>
          <p><strong>备注/取件码：</strong>{{item.note || '无'}}</p>
        </div>
      </div>
    </section>

    <!-- 🔹订单管理后台（仅你可见，可修改状态、价格） -->
    <section v-if="isAdmin" id="admin" class="section admin">
      <h2 class="title">📋 全部订单管理后台</h2>
      <div class="admin-box">
        <div v-if="orderList.length === 0" class="empty">暂无订单，等待顾客下单</div>
        <div v-for="(item, index) in orderList" :key="index" class="order-item">
          <p><strong>姓名：</strong>{{item.name}}</p>
          <p><strong>电话：</strong>{{item.phone}}</p>
          <p><strong>地址：</strong>{{item.address}}</p>
          <p><strong>服务：</strong>{{item.typeName}}｜{{item.deliverName}}</p>
          <p><strong>预估价格：</strong>{{item.price}} 元</p>
          <p><strong>实际价格：</strong><input v-model="item.realPrice" placeholder="送达后填写实际价"> 元</p>
          <p><strong>需补差价：</strong>{{Math.max(Number(item.realPrice)-Number(item.price), 0)}} 元</p>
          <p><strong>订单状态：</strong>
            <select v-model="item.status">
              <option value="wait">待取件</option>
              <option value="send">配送中</option>
              <option value="finish">已送达</option>
            </select>
          </p>
          <p><strong>备注/取件码：</strong>{{item.note || '无'}}</p>
          <button @click="deleteOrder(index)" class="del-btn">删除订单</button>
        </div>
      </div>
    </section>

    <!-- 首页轮播图 -->
    <section id="home" class="swiper">
      <div class="swiper-box">
        <div class="swiper-item" v-for="(item,index) in swiperList" :key="index" :class="{active:swiperIndex===index}">
          <div class="swiper-text">{{item.text}}</div>
        </div>
        <div class="swiper-dot">
          <span v-for="(item,index) in swiperList" :key="index" :class="{active:swiperIndex===index}" @click="swiperIndex=index"></span>
        </div>
      </div>
    </section>

    <!-- 服务介绍 -->
    <section id="service" class="section service">
      <h2 class="title">我们的服务</h2>
      <div class="card-group">
        <div class="card">
          <div class="card-icon">📦</div>
          <h3>快递代取</h3>
          <p>驿站/快递柜代取，可送楼下/楼上，支持取件码拍照识别</p>
        </div>
        <div class="card">
          <div class="card-icon">✉️</div>
          <h3>代寄快递</h3>
          <p>上门取件，协助打包、填写面单</p>
        </div>
        <div class="card">
          <div class="card-icon">🚪</div>
          <h3>大件搬运</h3>
          <p>重物/大件包裹，送达后补差价</p>
        </div>
      </div>
    </section>

    <!-- 收费标准 -->
    <section id="price" class="section price">
      <h2 class="title">收费标准</h2>
      <div class="price-table">
        <table>
          <thead>
            <tr>
              <th>快递类型</th>
              <th>送到楼下</th>
              <th>送到楼上</th>
              <th>备注说明</th>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td>中小件（≤2kg）</td>
              <td>1.5 元</td>
              <td>2.0 元</td>
              <td>固定价格，无差价</td>
            </tr>
            <tr>
              <td>大件（＞2kg）</td>
              <td>2.0 元起</td>
              <td>2.5–5 元</td>
              <td>预估2.5元，送达按重量补差价</td>
            </tr>
            <tr>
              <td>代寄快递</td>
              <td>2 元服务费</td>
              <td>—</td>
              <td>快递费顾客自理</td>
            </tr>
          </tbody>
        </table>
      </div>
    </section>

    <!-- 下单表单（新增取件码照片识别） -->
    <section id="order" class="section order">
      <h2 class="title">快速下单</h2>
      <div class="order-form">
        <input v-model="form.name" placeholder="请输入姓名" />
        <input v-model="form.phone" placeholder="请输入手机号" />
        <input v-model="form.address" placeholder="取件地址（自动回填宿舍信息，可修改）" />

        <select v-model="form.type" @change="calcPrice">
          <option value="">选择服务类型</option>
          <option value="small">中小件代取(≤2kg)</option>
          <option value="big">大件代取(＞2kg)</option>
          <option value="send">代寄快递</option>
        </select>

        <select v-model="form.deliver" @change="calcPrice">
          <option value="">选择配送方式</option>
          <option value="floor">送到楼下</option>
          <option value="up">送到楼上</option>
        </select>

        <div class="price-show" v-if="form.price">
          💰 预估应付：<strong>{{form.price}} 元</strong>
          <div v-if="form.type==='big'" class="tip">⚠️ 大件送达后按实际重量补差价</div>
        </div>

        <input v-model="form.time" placeholder="期望送达时间（例：18:00，服务9:00–20:30）" />

        <!-- 🔹取件码图片上传+识别 -->
        <div class="ocr-box">
          <label class="upload-btn">
            📷 上传取件码照片识别
            <input type="file" accept="image/*" @change="handleUpload" hidden>
          </label>
          <div v-if="ocrText" class="ocr-result">识别取件码：<strong>{{ocrText}}</strong></div>
        </div>

        <textarea v-model="form.note" placeholder="备注：取件码、快递位置、特殊要求">{{ocrText}}</textarea>

        <button class="btn-primary" @click="submitOrder">提交订单</button>
      </div>
    </section>

    <!-- 用户评价模块 -->
    <section id="comment" class="section comment">
      <h2 class="title">顾客真实评价</h2>
      <div class="comment-input">
        <input v-model="newComment.name" placeholder="您的昵称" />
        <textarea v-model="newComment.content" placeholder="写下您的评价，感谢支持！"></textarea>
        <button class="btn-primary" @click="addComment">提交评价</button>
      </div>
      <div class="comment-list">
        <div v-if="commentList.length===0" class="empty">暂无评价，期待您的反馈</div>
        <div v-for="(item,index) in commentList" :key="index" class="comment-item">
          <div class="comment-name">{{item.name}}</div>
          <div class="comment-time">{{item.time}}</div>
          <div class="comment-text">{{item.content}}</div>
        </div>
      </div>
    </section>

    <!-- 联系我们（服务时间：9:00–20:30） -->
    <section id="contact" class="section contact">
      <h2 class="title">联系我们</h2>
      <div class="contact-card">
        <p>📞 联系电话：<strong>15080910860</strong></p>
        <p>💬 咨询微信：<strong>同手机号 15080910860</strong></p>
        <p>📍 服务范围：本校全校区</p>
        <p>⏰ 服务时间：<strong>9:00 – 20:30</strong></p>
      </div>
    </section>

    <!-- 底部 -->
    <footer class="footer">
      <p>© 2025 校园快递代取 | 便捷 · 安心 · 靠谱 | 微信/电话：15080910860 | 服务时间9:00–20:30</p>
    </footer>
  </div>
</template>

<script setup>
import { ref, onMounted, watch, computed } from 'vue'

// ==========个人信息模块==========
const userInfo = ref(JSON.parse(localStorage.getItem('userInfo')) || {
  nickname: '',
  building: '',
  room: ''
})
const showUserPopup = ref(false)
onMounted(() => {
  if (!userInfo.value.nickname && !userInfo.value.building && !userInfo.value.room) {
    showUserPopup.value = true
  }
})
const saveUserInfo = () => {
  if (!userInfo.value.nickname || !userInfo.value.building || !userInfo.value.room) {
    alert('请填写完整昵称、楼栋、门牌号！')
    return
  }
  localStorage.setItem('userInfo', JSON.stringify(userInfo.value))
  showUserPopup.value = false
  form.value.address = `${userInfo.value.building}${userInfo.value.room}`
  alert('信息保存成功！')
}

// ==========轮播图模块==========
const swiperIndex = ref(0)
const swiperList = ref([
  { text: "校园快递代取｜足不出户，快递送到手边" },
  { text: "大件代取｜预估价格，送达后补差价" },
  { text: "取件码拍照识别｜全校区覆盖，9:00-20:30服务" }
])
onMounted(() => {
  setInterval(() => {
    swiperIndex.value = (swiperIndex.value + 1) % swiperList.value.length
  }, 4000)
})

// ==========取件码OCR识别模块（纯前端）==========
const ocrText = ref('')
const handleUpload = async (e) => {
  const file = e.target.files[0]
  if (!file) return
  // 浏览器内置OCR识别
  if (!window.ocr) {
    alert('请使用新版Chrome/Edge浏览器，支持图片文字识别')
    return
  }
  try {
    const result = await window.ocr.recognize(file)
    // 正则提取纯数字取件码
    const code = result.text.match(/\d{4,8}/)?.[0] || ''
    ocrText.value = code
    form.value.note = code
    alert(`识别成功！取件码：${code}`)
  } catch (err) {
    alert('识别失败，请上传清晰取件码照片')
  }
}

// ==========订单&表单模块==========
const isAdmin = ref(false)
const form = ref({
  name: userInfo.value.nickname || '',
  phone: '',
  address: `${userInfo.value.building}${userInfo.value.room}` || '',
  type: '',
  deliver: '',
  price: 0,
  time: '',
  note: ''
})
watch(userInfo, () => {
  form.value.address = `${userInfo.value.building}${userInfo.value.room}`
  form.value.name = userInfo.value.nickname
}, { deep: true })

// 订单、评价本地存储
const orderList = ref(JSON.parse(localStorage.getItem('orderList')) || [])
// 顾客自己的订单（筛选手机号匹配）
const myOrderList = computed(() => {
  return orderList.value.filter(item => item.phone === form.value.phone)
})
const commentList = ref(JSON.parse(localStorage.getItem('commentList')) || [])
const newComment = ref({ name: userInfo.value.nickname || '', content: '' })

// 自动计算预估价格
const calcPrice = () => {
  let p = 0
  const t = form.value.type
  const d = form.value.deliver
  if (t === 'small') {
    p = d === 'floor' ? 1.5 : 2
  } else if (t === 'big') {
    p = d === 'floor' ? 2 : 2.5
  } else if (t === 'send') {
    p = 2
  }
  form.value.price = p
}

// 提交订单（默认状态：待取件）
const submitOrder = () => {
  if (!form.value.name || !form.value.phone || !form.value.address || !form.value.type || !form.value.deliver) {
    alert('请填写完整信息！')
    return
  }
  const typeName = form.value.type === 'small' ? '中小件代取' : form.value.type === 'big' ? '大件代取' : '代寄快递'
  const deliverName = form.value.deliver === 'floor' ? '送到楼下' : '送到楼上'
  const newOrder = {
    ...form.value,
    typeName,
    deliverName,
    realPrice: form.value.price,
    status: 'wait' // wait待取件 send配送中 finish已送达
  }
  orderList.value.push(newOrder)
  localStorage.setItem('orderList', JSON.stringify(orderList.value))
  alert('下单成功！我们会尽快联系您～\n微信/电话：15080910860\n服务时间：9:00–20:30\n可在【我的订单】查看进度')
  form.value = { 
    name: userInfo.value.nickname || '', 
    phone: form.value.phone, 
    address: `${userInfo.value.building}${userInfo.value.room}` || '', 
    type: '', 
    deliver: '', 
    price: 0, 
    time: '', 
    note: '' 
  }
  ocrText.value = ''
}

// 删除订单
const deleteOrder = (index) => {
  orderList.value.splice(index, 1)
  localStorage.setItem('orderList', JSON.stringify(orderList.value))
}

// 提交评价
const addComment = () => {
  if (!newComment.value.name || !newComment.value.content) {
    alert('请填写昵称和评价内容！')
    return
  }
  const time = new Date().toLocaleString()
  commentList.value.push({ ...newComment.value, time })
  localStorage.setItem('commentList', JSON.stringify(commentList.value))
  newComment.value = { name: userInfo.value.nickname || '', content: '' }
}

// 平滑滚动
const goTo = (id) => {
  document.querySelector(id).scrollIntoView({ behavior: 'smooth' })
}
</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}
.container {
  background: #f7f9fc;
  color: #333;
}

/* ==========首次进入弹窗========== */
.user-popup {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0,0,0,0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 9999;
}
.popup-box {
  background: white;
  padding: 40px 30px;
  border-radius: 16px;
  width: 400px;
  display: flex;
  flex-direction: column;
  gap: 16px;
}
.popup-box h3 {
  color: #4a6fa5;
  text-align: center;
  font-size: 22px;
}
.popup-box .tip {
  color: #777;
  font-size: 13px;
  text-align: center;
}
.popup-box input {
  padding: 14px;
  border: 1px solid #e4e9f2;
  border-radius: 10px;
  font-size: 15px;
}

/* ==========导航========== */
.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 18px 6%;
  background: #ffffff;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.04);
  position: sticky;
  top: 0;
  z-index: 100;
}
.logo {
  font-size: 20px;
  font-weight: 600;
  color: #4a6fa5;
}
.nav a {
  margin-left: 22px;
  text-decoration: none;
  color: #555;
  font-size: 14px;
  transition: 0.3s;
}
.nav a:hover {
  color: #4a6fa5;
}

/* ==========轮播图========== */
.swiper {
  height: 320px;
  background: linear-gradient(135deg, #6b8ac2, #4a6fa5);
  position: relative;
  overflow: hidden;
}
.swiper-item {
  position: absolute;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  font-size: 28px;
  opacity: 0;
  transition: opacity 0.8s;
}
.swiper-item.active {
  opacity: 1;
}
.swiper-dot {
  position: absolute;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  gap: 10px;
}
.swiper-dot span {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  background: rgba(255,255,255,0.4);
  cursor: pointer;
}
.swiper-dot span.active {
  background: white;
}

/* ==========通用区块========== */
.section {
  padding: 70px 6%;
  text-align: center;
}
.title {
  font-size: 28px;
  color: #4a6fa5;
  margin-bottom: 40px;
  font-weight: 600;
}

/* ==========设置页面========== */
.setting-box {
  max-width: 420px;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  gap: 16px;
  background: white;
  padding: 30px;
  border-radius: 16px;
  box-shadow: 0 6px 20px rgba(0,0,0,0.05);
}
.setting-box input {
  padding: 14px;
  border: 1px solid #e4e9f2;
  border-radius: 10px;
  font-size: 15px;
}

/* ==========我的订单========== */
.myorder-box {
  max-width: 700px;
  margin: 0 auto;
  text-align: left;
}
.myorder-item {
  background: white;
  padding: 20px;
  border-radius: 12px;
  margin-bottom: 16px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.04);
}
.status-tag {
  padding: 4px 10px;
  border-radius: 6px;
  font-size: 13px;
  color: white;
}
.status-tag.wait { background:#f39c12; }
.status-tag.send { background:#3498db; }
.status-tag.finish { background:#27ae60; }

/* ==========取件码识别区域========== */
.ocr-box {
  display: flex;
  flex-direction: column;
  gap: 10px;
  text-align: left;
}
.upload-btn {
  padding: 12px;
  background: #eef2fb;
  border: 1px dashed #4a6fa5;
  border-radius: 10px;
  cursor: pointer;
  color: #4a6fa5;
  font-size: 15px;
}
.ocr-result {
  font-size: 14px;
  color: #27ae60;
}

/* ==========服务卡片========== */
.card-group {
  display: flex;
  justify-content: center;
  gap: 30px;
  flex-wrap: wrap;
}
.card {
  width: 280px;
  background: white;
  padding: 36px 20px;
  border-radius: 16px;
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.05);
  transition: 0.3s;
}
.card:hover {
  transform: translateY(-6px);
}
.card-icon {
  font-size: 36px;
  margin-bottom: 16px;
}
.card h3 {
  font-size: 20px;
  color: #4a6fa5;
  margin-bottom: 10px;
}
.card p {
  color: #666;
  font-size: 14px;
}

/* ==========价格表格========== */
.price-table {
  max-width: 780px;
  margin: 0 auto;
}
table {
  width: 100%;
  border-collapse: collapse;
  background: white;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.05);
}
th {
  background: #f0f4fa;
  padding: 16px;
  color: #4a6fa5;
}
td {
  padding: 16px;
  border-bottom: 1px solid #f5f7fa;
}

/* ==========下单表单========== */
.order-form {
  max-width: 520px;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  gap: 16px;
}
.order-form input,
.order-form select,
.order-form textarea {
  padding: 14px;
  border: 1px solid #e4e9f2;
  border-radius: 10px;
  font-size: 15px;
  background: white;
}
.order-form textarea {
  min-height: 100px;
  resize: none;
}
.price-show {
  font-size: 17px;
  color: #4a6fa5;
}
.tip {
  font-size: 13px;
  color: #e67e22;
  margin-top: 4px;
}
.btn-primary {
  padding: 14px;
  background: #4a6fa5;
  color: white;
  border: none;
  border-radius: 10px;
  font-size: 16px;
  cursor: pointer;
  transition: 0.3s;
}
.btn-primary:hover {
  background: #3d5d8a;
}

/* ==========评价模块========== */
.comment-input {
  max-width: 520px;
  margin: 0 auto 40px;
  display: flex;
  flex-direction: column;
  gap: 12px;
}
.comment-input input,
.comment-input textarea {
  padding: 12px;
  border: 1px solid #e4e9f2;
  border-radius: 10px;
}
.comment-list {
  max-width: 700px;
  margin: 0 auto;
  text-align: left;
}
.comment-item {
  background: white;
  padding: 20px;
  border-radius: 12px;
  margin-bottom: 16px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.04);
}
.comment-name {
  font-weight: 600;
  color: #4a6fa5;
}
.comment-time {
  font-size: 12px;
  color: #999;
  margin-bottom: 8px;
}
.comment-text {
  color: #444;
}
.empty {
  color: #999;
  padding: 30px;
}

/* ==========联系卡片========== */
.contact-card {
  max-width: 420px;
  margin: 0 auto;
  background: white;
  padding: 30px;
  border-radius: 16px;
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.05);
  text-align: left;
}
.contact-card p {
  margin: 10px 0;
  font-size: 16px;
  color: #444;
}

/* ==========订单后台========== */
.admin-box {
  max-width: 800px;
  margin: 0 auto;
  text-align: left;
}
.order-item {
  background: white;
  padding: 20px;
  border-radius: 12px;
  margin-bottom: 16px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.04);
}
.order-item input, .order-item select {
  padding: 4px 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
  margin-left: 8px;
}
.del-btn {
  margin-top: 10px;
  padding: 6px 12px;
  background: #e74c3c;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
}

/* ==========底部========== */
.footer {
  background: #4a6fa5;
  color: white;
  text-align: center;
  padding: 26px;
  font-size: 14px;
}
</style>