<template>
    <div id="productModal" ref="productModal" class="modal fade" tabindex="-1"
        aria-labelledby="productModalLabel"
           aria-hidden="true">
        <div class="modal-dialog modal-xl">
          <div class="modal-content border-0">
            <div class="modal-header bg-dark text-white">
              <h5 id="productModalLabel" class="modal-title">
                <span v-if="isNew">新增產品</span>
                <span v-else>編輯產品</span>
              </h5>
              <button type="button" class="btn-close"
              data-bs-dismiss="modal" aria-label="Close"></button>
            </div>
            <div class="modal-body">
              <div class="row">
                <div class="col-sm-4">
                  <div class="mb-2">
                    <h5>主要圖片</h5>
                    <div class="mb-3">
                      <label for="imageUrl" class="form-label">輸入圖片網址</label>
                      <input type="text" class="form-control"
                             placeholder="請輸入圖片連結" v-model="tempProduct.imageUrl">
                    </div>
                    <img class="img-fluid" :src="tempProduct.imageUrl" alt="">
                  </div>
                  <div class="mb-3">
                    <label for="customFile" class="form-label"
                      >或 上傳圖片
                      <i class="fas fa-spinner fa-spin"></i>
                    </label>
                    <input
                      type="file"
                      id="customFile"
                      class="form-control"
                      ref="fileInput"
                      @change="uploadFile"
                    />
                    <img class="img-fluid" :src="tempProduct.imageUrl" alt="" />
                  </div>
                  <h5>多圖新增</h5>
                  <div v-if="Array.isArray(tempProduct.imagesUrl)">
                    <div class="mb-1" v-for="(image, key) in tempProduct.imagesUrl" :key="key">
                      <div class="mb-3">
                        <label :for="`imagesUrl${key}`" class="form-label">圖片網址</label>
                        <input :id="`imagesUrl${key}`" v-model="tempProduct.imagesUrl[key]"
                        type="text" class="form-control"
                          placeholder="請輸入圖片連結">
                      </div>
                      <img class="img-fluid" :src="image">
                    </div>
                    <div v-if="!tempProduct.imagesUrl.length ||
                    tempProduct.imagesUrl[tempProduct.imagesUrl.length - 1]">
                      <button class="btn btn-outline-secondary btn-sm d-block w-100"
                        @click="tempProduct.imagesUrl.push('')">
                        新增圖片
                      </button>
                    </div>
                    <div v-else>
                      <button class="btn btn-outline-danger btn-sm d-block w-100"
                      @click="tempProduct.imagesUrl.pop()">
                        刪除圖片
                      </button>
                    </div>
                  </div>
                  <div v-else>
                    <button class="btn btn-outline-secondary btn-sm d-block w-100"
                    @click="createImages">
                      新增圖片
                    </button>
                  </div>
                </div>
                <div class="col-sm-8">
                  <div class="mb-3">
                    <label for="title" class="form-label">標題</label>
                    <input id="title" type="text" class="form-control" placeholder="請輸入標題"
                    v-model="tempProduct.title">
                  </div>

                  <div class="row">
                    <div class="mb-3 col-md-6">
                      <label for="category" class="form-label">分類</label>
                      <input id="category" type="text" class="form-control"
                             placeholder="請輸入分類" v-model="tempProduct.category">
                    </div>
                    <div class="mb-3 col-md-6">
                      <label for="unit" class="form-label">單位</label>
                      <input id="unit" type="text" class="form-control" placeholder="請輸入單位"
                      v-model="tempProduct.unit">
                    </div>
                  </div>

                  <div class="row">
                    <div class="mb-3 col-md-6">
                      <label for="origin_price" class="form-label">原價</label>
                      <input id="origin_price" type="number" min="0" class="form-control"
                      placeholder="請輸入原價" v-model.number="tempProduct.origin_price">
                    </div>
                    <div class="mb-3 col-md-6">
                      <label for="price" class="form-label">售價</label>
                      <input id="price" type="number" min="0" class="form-control"
                             placeholder="請輸入售價" v-model.number="tempProduct.price">
                    </div>
                  </div>
                  <hr>

                  <div class="mb-3">
                    <label for="description" class="form-label">產品描述</label>
                    <textarea id="description" type="text" class="form-control"
                              placeholder="請輸入產品描述" v-model="tempProduct.description">
                    </textarea>
                  </div>
                  <div class="mb-3">
                    <label for="content" class="form-label">說明內容</label>
                    <textarea id="description" type="text" class="form-control"
                              placeholder="請輸入說明內容" v-model="tempProduct.content">
                    </textarea>
                  </div>
                  <div class="row">
                  <div class="mb-3 col-md-6">
                    <div class="form-check">
                      <input id="is_enabled" class="form-check-input" type="checkbox"
                             :true-value="1" :false-value="0" v-model="tempProduct.is_enabled">
                      <label class="form-check-label" for="is_enabled">是否上架</label>
                    </div>
                  </div>
                  <div class="mb-3 col-md-6">
                      <label class="form-label" for="star">網友推薦指數</label>
                      <input id="star" class="form-control" type="text" v-model="tempProduct.star">
                  </div>

                </div>
                </div>
              </div>
            </div>
            <div class="modal-footer">
              <button type="button" class="btn btn-outline-secondary" data-bs-dismiss="modal">
                取消
              </button>
              <button type="button" class="btn btn-primary" @click="updateData">
                確認
              </button>
            </div>
          </div>
        </div>
    </div>
</template>

<script>
import axios from 'axios';
import { Modal } from 'bootstrap';

const { VITE_API_URL, VITE_API_PATH } = import.meta.env;
export default {
  props: ['product', 'isNew'],
  data() {
    return {
      productModal: null,
      tempProduct: {},
    };
  },
  methods: {
    updateData() {
      let apiUrl = `${VITE_API_URL}/api/${VITE_API_PATH}/admin/product/${this.product.id}`;
      let http = 'put';
      if (this.isNew) {
        apiUrl = `${VITE_API_URL}/api/${VITE_API_PATH}/admin/product`;
        http = 'post';
      }
      axios[http](apiUrl, { data: this.product })
        .then(() => {
          // console.log(res.config.data);
          // alert(res.data.message);
          this.closeModal();
          this.$emit('update');
        })
        .catch(() => {
          // alert(err.data.message);
        });
    },
    uploadFile() {
      const uploadedFile = this.$refs.fileInput.files[0];
      const formData = new FormData();
      formData.append('file-to-upload', uploadedFile);
      axios
        .post(`${VITE_API_URL}/api/${VITE_API_PATH}/admin/upload`, formData)
        .then((res) => {
          this.tempProduct.imageUrl = res.data.imageUrl;
        })
        .catch(() => {
          // alert(err.data.message);
        });
    },
    createImages() {
      this.tempProduct.imagesUrl = [];
    },
    openModal() {
      this.productModal.show();
    },
    closeModal() {
      this.productModal.hide();
    },
  },
  watch: {
    product() {
      this.tempProduct = this.product;
    },
  },
  mounted() {
    this.productModal = new Modal(this.$refs.productModal);
    this.tempProduct = this.product;
  },
};
</script>
