<template>
  <div class="container">
    <div class="row register-page">
      <form class="col s12" id="reg-form">
        <div class="row">
          <div class="errorMessage">{{ errorMessageforDuplicateEmail }}</div>
          <div class="input-field col s6">
            <div class="errorMessage">{{ errorMessageforLastName }}</div>
            <input
              id="last_name"
              type="text"
              class="validate"
              v-model="lastName"
              required
            />
            <label for="last_name">姓</label>
          </div>
          <div class="input-field col s6">
            <div class="errorMessage">{{ errorMessageforFirstName }}</div>
            <input
              id="first_name"
              type="text"
              class="validate"
              v-model="firstName"
              required
            />
            <label for="first_name">名</label>
          </div>
        </div>
        <!-- 住所 -->

        <div class="row">
          <div class="input-field col s6">
            <!-- <div class="errorMessage">{{ errorMessageforAddress }}</div> -->
            <input
              id="zipCode"
              type="text"
              class="validate"
              size="7"
              v-model="zipCode"
              required
            />
            <label for="zipCode">郵便番号</label>
          </div>
          <div class="row">
            <button
              class="waves-effect waves-light btn searchBtn"
              id="get_address_btn"
              type="button"
              @click="getAddress()"
            >
              住所検索
            </button>
          </div>
        </div>
        <div class="row">
          <div class="input-field col s12">
            <input
              id="address"
              type="text"
              class="validate"
              v-model="address"
              required
            />
            <label for="address">住所</label>
          </div>
        </div>

        <!-- 住所ここまで -->
        <div class="row">
          <div class="input-field col s12">
            <div class="errorMessage">{{ errorMessageforEmail }}</div>
            <input
              id="email"
              type="email"
              class="validate"
              v-model="mailAddress"
              required
            />
            <label for="email">メールアドレス</label>
          </div>
        </div>
        <div class="row">
          <div class="input-field col s12">
            <div class="errorMessage">{{ errorMessageforPassword }}</div>
            <input
              id="password"
              type="password"
              class="validate"
              minlength="8"
              v-model="password"
              required
            />
            <label for="password">パスワード</label>
          </div>
        </div>
        <div class="row">
          <div class="input-field col s6">
            <button
              class="btn btn-large btn-register waves-effect waves-light"
              type="button"
              v-on:click="registerAdmin"
            >
              登録
              <i class="material-icons right">done</i>
            </button>
          </div>
        </div>
      </form>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import config from "@/const/const";
import axios from "axios";
// import axiosJsonpAdapter from "axios-jsonp";

/**
 * 管理者登録をする画面.
 */
@Component
export default class RegisterAdmin extends Vue {
  // 姓
  private lastName = "";
  // 名
  private firstName = "";
  // 郵便番号
  private zipCode = "";
  // 住所
  private address = "";
  // メールアドレス
  private mailAddress = "";
  // パスワード
  private password = "";
  // 姓のエラーメッセージ
  private errorMessageforLastName = "";
  // 名のエラーメッセージ
  private errorMessageforFirstName = "";
  // メールアドレスのエラーメッセージ
  private errorMessageforEmail = "";
  // パスワードのエラーメッセージ
  private errorMessageforPassword = "";
  // 重複メールアドレスのエラーメッセージ
  private errorMessageforDuplicateEmail = "";
  /**
   * 管理者情報を登録する.
   *
   * @remarks
   * 本メソッドは非同期でWebAPIを呼び出し管理者登録をするためasync/await axiosを利用しています。これらを利用する場合は明示的に戻り値にPromiseオブジェクト型を指定する必要があります。
   * @returns Promiseオブジェクト
   */
  async registerAdmin(): Promise<void> {
    let hasErrors = false;
    if (this.lastName == "") {
      this.errorMessageforLastName = "姓を入力してください";
      hasErrors = true;
    }
    if (this.firstName == "") {
      this.errorMessageforFirstName = "名を入力してください";
      hasErrors = true;
    }
    if (this.mailAddress == "") {
      this.errorMessageforEmail = "メールアドレスを入力してください";
      hasErrors = true;
    }
    if (this.password == "") {
      this.errorMessageforPassword = "パスワードを入力してください";
      hasErrors = true;
    }
    if (hasErrors) {
      return;
    }
    // 管理者登録処理
    const response = await axios.post(`${config.EMP_WEBAPI_URL}/insert`, {
      name: this.lastName + " " + this.firstName,
      mailAddress: this.mailAddress,
      password: this.password,
      address: this.address,
    });
    console.dir("response:" + JSON.stringify(response));
    if (response.data.status == "success") {
      this.$router.push("/loginAdmin");
    } else if (response.data.status == "error") {
      this.errorMessageforDuplicateEmail = "登録できませんでした";
    }
  }
  async getAddress(): Promise<void> {
    // eslint-disable-next-line @typescript-eslint/no-var-requires
    const axiosJsonpAdapter = require("axios-jsonp");
    const response = await axios.get("https://zipcoda.net/api", {
      adapter: axiosJsonpAdapter,
      params: {
        zipcode: this.zipCode,
      },
    });
    console.dir(JSON.stringify(response));
    this.address = response.data.items[0].address;
  }
}
</script>

<style scoped>
.register-page {
  width: 600px;
}

.errorMessage {
  color: red;
}

.searchBtn {
  margin: 10px;
}
</style>
