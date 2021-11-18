<template>
  <div class="container">
    <div class="row register-page">
      <form class="col s12" id="reg-form">
        <div class="row">
          <div class="errorMessage">{{ errorMessageforDuplicateEmail }}</div>
          <div class="errorMessage">{{ errorMessageforLastName }}</div>
          <div class="errorMessage">{{ errorMessageforFirstName }}</div>
          <div class="input-field col s6">
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
        <div class="row">
          <div class="errorMessage">{{ errorMessageforEmail }}</div>
          <div class="input-field col s12">
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
          <div class="errorMessage">{{ errorMessageforPassword }}</div>
          <div class="errorMessage">
            {{ errorMessageforConfirmationPassword }}
          </div>
          <div class="input-field col s12">
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
          <div class="input-field col s12">
            <div class="errorMessage"></div>
            <input
              id="confirmationPassword"
              type="password"
              class="validate"
              minlength="8"
              v-model="confirmationPassword"
              required
            />
            <label for="confirmationPassword">確認用パスワード</label>
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

/**
 * 管理者登録をする画面.
 */
@Component
export default class RegisterAdmin extends Vue {
  // 姓
  private lastName = "";
  // 名
  private firstName = "";
  // メールアドレス
  private mailAddress = "";
  // パスワード
  private password = "";
  // 確認用パスワード
  private confirmationPassword = "";
  // 姓のエラーメッセージ
  private errorMessageforLastName = "";
  // 名のエラーメッセージ
  private errorMessageforFirstName = "";
  // メールアドレスのエラーメッセージ
  private errorMessageforEmail = "";
  // パスワードのエラーメッセージ
  private errorMessageforPassword = "";
  // 確認用パスワードのエラーメッセージ
  private errorMessageforConfirmationPassword = "";
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
    if (this.confirmationPassword == "") {
      this.errorMessageforConfirmationPassword =
        "確認用パスワードを入力してください";
      hasErrors = true;
    } else if (this.password !== this.confirmationPassword) {
      this.errorMessageforConfirmationPassword = "パスワードが一致していません";
      hasErrors = true;
      this.password = "";
      this.confirmationPassword = "";
    }
    if (hasErrors) {
      return;
    }
    // 管理者登録処理
    const response = await axios.post(`${config.EMP_WEBAPI_URL}/insert`, {
      name: this.lastName + " " + this.firstName,
      mailAddress: this.mailAddress,
      password: this.password,
    });
    console.dir("response:" + JSON.stringify(response));
    if (response.data.status == "success") {
      this.$router.push("/loginAdmin");
    } else if (response.data.status == "error") {
      this.errorMessageforDuplicateEmail = "登録できませんでした";
    }
  }
}
</script>

<style scoped>
.register-page {
  width: 600px;
}

.errorMessage {
  color: red;
  margin-left: 10px;
}
</style>
