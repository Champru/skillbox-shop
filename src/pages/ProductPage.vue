<template>
  <!-- Прелоадер -->
  <AppPreloader v-if="productLoading" class="full-screen-height" />
  <!-- Ошибка при загрузке -->
  <ProductsLoadingFalled
    v-else-if="productLoadingFaled"
    btn-title="Назад"
    :action="goBack"
    class="full-screen-height"
  />
  <main class="content container" v-else>
    <div class="content__top">
      <ul class="breadcrumbs">
        <li class="breadcrumbs__item">
          <router-link class="breadcrumbs__link" :to="{name: 'main'}">
            Каталог
          </router-link>
        </li>
        <li class="breadcrumbs__item">
          <router-link class="breadcrumbs__link" :to="{name: 'main'}">
           {{ category.title }}
          </router-link>
        </li>
        <li class="breadcrumbs__item">
          <a class="breadcrumbs__link">
            {{ product.title }}
          </a>
        </li>
      </ul>
    </div>

    <section class="item">
      <div class="item__pics pics">
        <div class="pics__wrapper">
          <img width="570" height="570" :src="product.img" :alt="product.title">
        </div>
      </div>

      <div class="item__info">
        <span class="item__code">Артикул: {{ product.id }}</span>
        <h2 class="item__title">
          {{ product.title }}
        </h2>
        <div class="item__form">
          <form class="form" action="#" @submit.prevent="addToCart">
            <b class="item__price">
              {{ product.price | numberFormat}} ₽
            </b>

            <fieldset class="form__block">
              <legend class="form__legend">Цвет:</legend>
              <!-- Выбор цветов -->
              <ColorsList
                :colors="colors"
                :elementName="'product_' + product.id"
                :current-color="firstColor"
              />
            </fieldset>

            <fieldset class="form__block">
              <legend class="form__legend">Объемб в ГБ:</legend>

              <ul class="sizes sizes--primery">
                <li class="sizes__item">
                  <label class="sizes__label">
                    <input class="sizes__radio sr-only" type="radio" name="sizes-item" value="32">
                    <span class="sizes__value">
                      32gb
                    </span>
                  </label>
                </li>
                <li class="sizes__item">
                  <label class="sizes__label">
                    <input class="sizes__radio sr-only" type="radio" name="sizes-item" value="64">
                    <span class="sizes__value">
                      64gb
                    </span>
                  </label>
                </li>
                <li class="sizes__item">
                  <label class="sizes__label">
                    <input class="sizes__radio sr-only" type="radio" name="sizes-item" value="128" checked="">
                    <span class="sizes__value">
                      128gb
                    </span>
                  </label>
                </li>
              </ul>
            </fieldset>

            <div class="item__row">
              <!-- Счетчик -->
              <AppCounter v-model.number="productAmount" />
              <button class="button button--primery" type="submit" :disabled="productAddSending">
                В корзину
              </button>
            </div>
            <!-- Прелоадер -->
            <AppPreloader v-if="productAddSending" class="loader--small small-top-indent" />
            <div class="small-top-indent" v-show="productAdded">Товар добавлен в корзину</div>
          </form>
        </div>
      </div>

      <div class="item__desc">
        <ul class="tabs">
          <li class="tabs__item">
            <a class="tabs__link tabs__link--current">
              Описание
            </a>
          </li>
          <li class="tabs__item">
            <a class="tabs__link" href="#">
              Характеристики
            </a>
          </li>
          <li class="tabs__item">
            <a class="tabs__link" href="#">
              Гарантия
            </a>
          </li>
          <li class="tabs__item">
            <a class="tabs__link" href="#">
              Оплата и доставка
            </a>
          </li>
        </ul>

        <div class="item__content">
       

          <a href="#">
            Все характеристики
          </a>

          <h3>Что это?</h3>

          <p>
Lorem ipsum dolor sit amet, consectetur adipisicing elit. Cumque aperiam, soluta ullam temporibus fugiat quia repudiandae expedita voluptas fuga obcaecati culpa sapiente optio incidunt enim maiores provident nisi totam aspernatur?          </p>

          <h3>Дизайн</h3>

          <p>
Lorem ipsum dolor, sit amet consectetur adipisicing elit. Eligendi, doloribus sequi? Eum deleniti, iusto labore omnis nulla perspiciatis laboriosam commodi, blanditiis neque cumque sunt rerum fugiat? Saepe atque vero magnam.          </p>
        </div>
      </div>
    </section>
  </main>
</template>

<script>
import axios from 'axios';
import { API_BASE_URL } from '@/config';
import { mapActions, mapMutations } from 'vuex';
import ColorsList from '@/components/colors/ColorsList.vue';
import AppCounter from '@/components/common/AppCounter.vue';
import ProductsLoadingFalled from '@/components/product/ProductsLoadingFalled.vue';
import AppPreloader from '@/components/common/AppPreloader.vue';
import numberFormat from '@/helpers/numberFormat';

export default {
  components: {
    ColorsList,
    AppCounter,
    AppPreloader,
    ProductsLoadingFalled,
  },
  data() {
    return {
      productAmount: 1,
      productData: null,

      productLoading: false,
      productLoadingFaled: false,

      productAdded: false,
      productAddSending: false,
    };
  },
  filters: {
    numberFormat,
  },
  methods: {
    ...mapActions(['addProductToCart']),
    ...mapMutations(['productAddSendingStatus']),

    addToCart() {
      if (this.productAmount > 0 && Number.isInteger(this.productAmount)) {
        this.productAdded = false;
        this.productAddSending = true;

        this.addProductToCart({ productId: this.product.id, amount: this.productAmount })
          .then(() => {
            this.productAdded = true;
            this.productAddSending = false;
          });
      }
    },
    loadProduct() {
      this.productLoading = true;
      this.productLoadingFaled = false;
      setTimeout(() => {
        axios.get(API_BASE_URL + '/api/products/' + this.$route.params.id)
          .then((response) => {
            this.productData = response.data;
            this.productData.img = this.productData.image.file.url;
          })
          .catch(() => { this.productLoadingFaled = true; })
          .then(() => { this.productLoading = false; });
      }, 1000);
    },
    goBack() {
      this.$router.go(-1);
    },
  },
  computed: {
    colors() {
      return this.productData.colors;
    },
    firstColor() {
      return this.colors[0].id;
    },
    product() {
      return this.productData;
    },
    category() {
      return this.productData.category;
    },
  },
  watch: {
    '$route.params.id': {
      handler() {
        this.loadProduct();
      },
      immediate: true,
    },
    productAddSending() {
      this.productAddSendingStatus(this.productAddSending);
    },
  },
};
</script>

<style lang="stylus">
.small-top-indent
  margin-top 15px

.full-screen-height
  min-height 100vh

</style>
