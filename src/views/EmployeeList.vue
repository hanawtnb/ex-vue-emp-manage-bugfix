<template>
  <div class="container">
    <!-- パンくずリスト -->
    <nav>
      <div class="nav-wrapper">
        <div class="col s12 teal">
          <a class="breadcrumb">従業員リスト</a>
        </div>
      </div>
    </nav>
    <div>従業員数:{{ getEmployeeCount }}人</div>
    <!-- 従業員検索 -->
    <form class="searchForm">
      <input type="text" v-model="searchName" />
    </form>
    <button
      class="waves-effect waves-light btn searchBtn"
      type="button"
      @click="searchEmployeeByName"
    >
      検索
    </button>
    <div class="errorMessage">{{ errorMessageforNoHit }}</div>

    <div class="row">
      <table class="striped">
        <thead>
          <tr>
            <th>名前</th>
            <th>入社日</th>
            <th>扶養人数</th>
          </tr>
        </thead>

        <tbody>
          <tr v-for="employee of employees" v-bind:key="employee.id">
            <td>
              <router-link :to="'/employeeDetail/' + employee.id">{{
                employee.name
              }}</router-link>
            </td>
            <td>{{ employee.formatHireDate }}</td>
            <td>{{ employee.dependentsCount }}人</td>
          </tr>
        </tbody>
      </table>
    </div>
    <div class="paging">
      <span v-for="page of pageNumber" v-bind:key="page">
        <button
          class="waves-effect waves-light btn"
          type="button"
          @click="onMovePage(page)"
        >
          {{ page }}</button
        >&nbsp;
      </span>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import { Employee } from "@/types/employee";
/**
 * 従業員一覧を表示する画面.
 */
@Component
export default class EmployeeList extends Vue {
  // 従業員一覧
  private currentEmployeeList: Array<Employee> = [];
  // 従業員数
  private employeeCount = 0;
  // 最初に表示されるページ数
  private page = 1;
  // 検索する従業員名
  private searchName = "";
  // 曖昧検索で1件も見つからなかった時にエラーメッセージ
  private errorMessageforNoHit = "";

  /**
   * Vuexストアのアクション経由で非同期でWebAPIから従業員一覧を取得する.
   *
   * @remarks
   * Vueインスタンスが生成されたタイミングで
   * Vuexストア内のアクションを呼ぶ(ディスパッチする)。
   * ライフサイクルフックのcreatedイベント利用。
   *
   * 取得してからゲットするため、async awaitを利用している。
   */
  async created(): Promise<void> {
    // もしログイン状態がfalseならログイン画面に遷移する
    if (this["$store"].getters.getLoginStatus == false) {
      this.$router.push("/loginAdmin");
      return;
    }

    await this.$store.dispatch("getEmployeeList");

    // 従業員一覧情報をVuexストアから取得
    // 非同期で外部APIから取得しているので、async/await使わないとGetterで取得できない
    // ページング機能実装のため最初の10件に絞り込み
    this.currentEmployeeList = this["$store"].getters.getAllEmployees;
  }
  /**
   * 現在表示されている従業員一覧の数を返す.
   *
   * @returns 現在表示されている従業員一覧の数
   */
  get getEmployeeCount(): number {
    return this.currentEmployeeList.length;
  }
  /**
   * 全従業員数を返す.
   * @returns 従業員数
   */
  get allEmployeeCount(): number {
    return this["$store"].getters.getAllEmployeeCount;
  }
  /**
   * ページ数を計算して返す.
   *
   * @return 用意する必要のあるページ数
   */
  get pageNumber(): number {
    return Math.ceil(this.allEmployeeCount / 10);
  }
  /**
   * １ページで取得する従業員のリストを取得して返す.
   *
   * @remarks １ページで表示する人数は10人
   * @returns 1人目から10人ずつ切り取って返す
   */
  get employees(): Array<Employee> {
    const employee = this["$store"].getters.getAllEmployees;
    return employee.slice((this.page - 1) * 10, this.page * 10);
  }

  onMovePage(page: number): void {
    this.page = page;
  }
  
   * 従業員名で曖昧検索した結果を返す.
   *
   * @return 渡された文字列を含む従業員
   */
  searchEmployeeByName(): void {
    let hitEmployees = this["$store"].getters.getSearchEmployeeByName(
      this.searchName
    );
    let hasErrors = false;
    console.log("hitEmployees:" + hitEmployees);

    // 検索結果がなしの場合、未入力で検索された場合は全件を返す
    if (hitEmployees.length == 0 || this.searchName == "") {
      this.errorMessageforNoHit = "１件もありませんでしたので全件表示します";
      this.currentEmployeeList = this["$store"].getters.getAllEmployees;
      hasErrors = true;
    }
    if (hasErrors) {
      return;
    }
    // 検索した結果を返す
    this.errorMessageforNoHit = "";
    this.currentEmployeeList = this["$store"].getters.getSearchEmployeeByName(
      this.searchName
    );
  }
}
</script>

<style scoped>
.searchForm {
  margin-bottom: 20px;
  width: 450px;
  margin: 0 auto;
}

.searchBtn {
  display: block;
  width: 150px;
  margin: 0 auto;
}

.paging {
  text-align: center;
  padding-bottom: 30px;
}
.errorMessage {
  color: red;
  margin: 10px;
}
</style>
