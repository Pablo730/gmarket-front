<template>
  <v-container>
    <v-container
      class="d-flex justify-end ma-1 px-16">
      <base-write-post-button
        fromBoard="used">
      </base-write-post-button>
    </v-container>
    <v-card
      class="mx-auto"
      max-width="1000">
      <v-simple-table>
        <tbody>
          <tr>
            <td>
              검색어
            </td>
            <td
              colspan="2">
              <v-row
                class="mx-1 mt-2 pa-1">
                <v-text-field 
                  v-model="searchKeywordText"
                  label="검색어 입력"
                  @keyup.enter="changeSearchKeyword"
                  class="ma-auto mx-2 mb-0">
                </v-text-field>
                <v-btn
                  @click="changeSearchKeyword"
                  class="mx-auto mb-1"
                  color="primary">검색</v-btn>
              </v-row>
            </td>
          </tr>
          <tr>
            <td>카테고리</td>
            <td
              colspan="2">
              <v-select
                label="카테고리 선택"
                v-model="boardCategory"
                :items="$store.state.boardCategory">
              </v-select>
            </td>
          </tr>
          <tr>
            <td>판매상태</td>
            <td>
              <v-checkbox
                v-model="statusCreated">
                 <span slot="label" class="black--text">판매중</span>
              </v-checkbox>
            </td>
            <td>
              <v-checkbox
                v-model="statusClosed">
                 <span slot="label" class="black--text">판매종료</span>
              </v-checkbox>
            </td>
          </tr>
          <tr>
            <td>상품상태</td>
            <td>
              <v-checkbox
                v-model="usedGoodsStatusNew">
                 <span slot="label" class="black--text">새 것</span>
              </v-checkbox>
            </td>
            <td>
              <v-checkbox
                v-model="usedGoodsStatusUsed">
                 <span slot="label" class="black--text">사용감 있음</span>
              </v-checkbox>
            </td>
          </tr>
          <tr>
            <td>가격대 설정</td>
            <td>
              <v-text-field
                type="number"
                v-model="min"
                label="최소값 입력">
              </v-text-field>
            </td>
            <td>
              <v-text-field
                type="number"
                v-model="max"
                label="최대값 입력">
              </v-text-field>
            </td>
          </tr>
          <tr>
            <td>
              정렬
            </td>
            <td
              colspan="2">
              <div
                class="pl-10">
                <v-radio-group
                  v-model="sort"
                  row>
                  <v-radio
                    value="time">
                    <span slot="label" class="black--text">최신 순</span>
                  </v-radio>
                  <v-radio
                    value="LOW_PRICE">
                    <span slot="label" class="black--text">낮은 가격 순</span>
                  </v-radio>
                  <v-radio
                    value="HIGH_PRICE">
                    <span slot="label" class="black--text">높은 가격 순</span>
                  </v-radio>
                </v-radio-group>
              </div>
            </td>
          </tr>
          <tr>
            <td
              colspan="1">
              <v-btn
                class="my-3"
                color="indigo"
                @click="initSearch">
                초기화
              </v-btn>
            </td>
            <td
              colspan="2">
              <div
                v-if="searchKeyword !== ''"
                class="text-center">
                <div v-if="!isSearchKeywordInAlertKeywords">
                  <a
                    v-text="searchKeyword"
                    @click="addKeyword"></a>
                  <span> 키워드를 추가하시겠습니까?</span>
                </div>
                <div v-if="isSearchKeywordInAlertKeywords">
                  <a
                    v-text="searchKeyword"
                    @click="removeKeyword"></a>
                  <span> 키워드를 삭제하시겠습니까?</span>
                </div>
              </div>
              <v-snackbar
                v-model="snackbar"
                :timeout="timeout"
              >
                {{ snackbarText }}
                <template v-slot:action="{ attrs }">
                  <v-btn
                    color="blue"
                    text
                    v-bind="attrs"
                    @click="snackbar = false"
                  >
                    Close
                  </v-btn>
                </template>
              </v-snackbar>
            </td>
          </tr>
        </tbody>
      </v-simple-table>
    </v-card>
    <v-card
      max-width="1000"
      class="mx-auto py-2">
      <v-container
        class="d-flex flex-wrap">
        <base-item-card
          v-for="item in usedItems"
          :key="item.id"
          :board="item"/>
      </v-container>
      <v-divider class="my-1"/>
      <v-pagination
        v-model="page"
        :length="pageLength"
        :total-visible="10">
      </v-pagination>
    </v-card>
  </v-container>
</template>

<script>
export default {
  data: () => ({
    page: 1,
    pageLength: 1,
    snackbar: false,
    snackbarText: '',
    timeout: 1500,
    usedItems: [],
    alertKeyword: [],
    boardCategory: '',
    min: null,
    max: null,
    // time, lowPrice, highPrice
    sort: 'time',
    searchKeywordText: '',
    searchKeyword: '',
    // CREATED, CLOSED
    statusCreated: false,
    statusClosed: false,
    // NEW. USED
    usedGoodsStatusNew: false,
    usedGoodsStatusUsed: false,
  }),
  computed:{
    searchChangeChecker() {
      // computed에서 각 변수에 담긴 값을 확인하고 어떤 값이든지 변경사항이 생길경우 리턴함.
      const {
        page,
        min,
        max,
        sort,
        boardCategory,
        searchKeyword,
        statusCreated,
        statusClosed,
        usedGoodsStatusNew,
        usedGoodsStatusUsed,
      } = this

      return {
        page,
        min,
        max,
        sort,
        boardCategory,
        searchKeyword,
        statusCreated,
        statusClosed,
        usedGoodsStatusNew,
        usedGoodsStatusUsed,
      }
    },
    isSearchKeywordInAlertKeywords() {
      let result = false;
      this.alertKeyword.forEach(e => {
        if(e.keyword === this.searchKeyword) {
          result = true;
        }
      })
      return result;
    },
  },
  watch: {
    searchChangeChecker() {
      this.getUsedBoard();
    },
  },
  methods: {
    async getUsedBoard() {
      const APIURL = `${process.env.VUE_APP_API_URL}/boards/used/posts/search`

      let params = {
        page: this.page
      };

      if(this.boardCategory !== '') {
        params.category = this.boardCategory;
      }

      if(this.min === null && this.max !== null) {
        params.max = this.max;
      }

      if(this.min !== null && this.max === null) {
        params.min = this.min;
      }

      if(this.min !== null || this.max !== null) {
        params.min = this.min;
        params.max = this.max;
      }

      if(this.searchKeyword !== '') {
        params.title = this.searchKeyword;
      }

      if(this.statusCreated ^ this.statusClosed) {
        params.status = this.statusCreated ? 'CREATED' : 'CLOSED'
      }

      if(this.usedGoodsStatusNew ^ this.usedGoodsStatusUsed) {
        params.goodsStatus = this.usedGoodsStatusNew ? 'NEW' : 'USED'
      }

      if(this.sort !== 'time') {
        params.sort = this.sort;
      }

      try {
        const { data } = await this.$axios.get(APIURL, {params: params});
        this.usedItems = data.data.board_list;
        this.pageLength = data.data.total_page;
      } catch(err) {
        alert(`중고 게시판 조회 오류\n${err}`);
      }
    },
    initSearch() {
      this.sort = 'time';
      this.searchKeywordText = '';
      this.searchKeyword = '';
      this.boardCategory = '';
      this.statusCreated= false;
      this.statusClosed= false;
      this.usedGoodsStatusNew= false;
      this.usedGoodsStatusUsed= false;
      this.min = null;
      this.max = null;
    },
    changeSearchKeyword() {
      this.searchKeyword = this.searchKeywordText;
    },
    async addKeyword() {
      const APIURL = process.env.VUE_APP_API_URL;

      try {
        await this.$axios.post(`${APIURL}/alert-keyword`, {
          user_id: this.$store.state.userId,
          keyword: this.searchKeyword
        })
      } catch(err) {
        console.log(err);
      }
      this.snackbarText = '추가되었습니다.'
      this.snackbar = true;
      await this.getAlertKeyword();
      this.alertKeyword = this.$store.state.alertKeyword.data;
    },
    async removeKeyword() {
      const APIURL = process.env.VUE_APP_API_URL;

      let keywordId;
      
      this.alertKeyword.forEach(e => {
        if(e.keyword === this.searchKeyword) {
          keywordId = e.alert_keyword_id;
          return;
        }
      })

      try {
        await this.$axios.delete(`${APIURL}/alert-keyword?id=${keywordId}`)
      } catch(err) {
        console.log(err);
      }
      this.snackbarText = '삭제되었습니다.'
      this.snackbar = true;
      await this.getAlertKeyword();
      this.alertKeyword = this.$store.state.alertKeyword.data;
    },
    async getAlertKeyword() {
      try {
        await this.$store.dispatch('GETALERTKEYWORD', this.$store.state.userId);
      } catch(err) {
        console.log(err)
      }
    },
  },
  async mounted() {
    this.alertKeyword = this.$store.state.alertKeyword.data;
    await this.getUsedBoard();
  }
}
</script>