<template>
  <v-container>
    <v-card
      max-width="500"
      class="mx-auto my-12">
      <v-form v-model="valid">
        <v-container
          class="ma-5">
          <v-card-title class="ma-2 ml-4 mb-10">회원가입</v-card-title>
          <v-text-field
            label="아이디"
            required
            v-model="userId"
            outlined
            :rules="idRules"
            class="mt-2 mb-2 mr-4 pr-2"></v-text-field>
          <v-text-field
            label="비밀번호"
            v-model="password"
            required
            outlined
            :rules="pwRules"
            type="password"
            class="mt-2 mb-2 mr-4 pr-2"></v-text-field>
          <v-text-field
            label="비밀번호 확인"
            v-model="passwordCheck"
            required
            outlined
            :rules="pwCheckRules"
            type="password"
            class="mt-2 mb-2 mr-4 pr-2"></v-text-field>
          <v-text-field
            label="닉네임"
            v-model="nickName"
            required
            outlined
            :rules="nickRules"
            @keyup.enter="signUp"
            class="mt-2 mb-2 mr-4 pr-2"></v-text-field>
          <v-container
            class="d-flex justify-end">
            <v-btn
              :disabled="!valid"
              class="ma-4"
              color="success"
              @click="signUp">회원가입</v-btn>
          </v-container>
        </v-container>
      </v-form>
    </v-card>
  </v-container>
</template>

<script>
export default {
  data: () => ({
    valid: true,
    userId: '',
    password: '',
    passwordCheck: '',
    nickName: '',
    idRules: [
      value => !!value || '필수 입력값입니다.',
      value => (value && value.length >= 4) || '4자 이상 입력해주세요.',
      value => (value && value.length <= 25) || '25자 이하로 입력해주세요.',
    ],
    pwRules: [
      value => !!value || '필수 입력값입니다.',
      value => (value && value.length >= 6) || '6자 이상 입력해주세요.',
      value => (value && value.length <= 25) || '25자 이하로 입력해주세요.',
    ],
    pwCheckRules: [
      value => !!value || '필수 입력값입니다.',
    ],
    nickRules: [
      value => !!value || '필수 입력값입니다.',
      value => (value && value.length >= 2) || '2자 이상 입력해주세요'
    ]
  }),
  computed: {
    passwordConfirmMatcher() {
      return this.password === this.passwordCheck;
    }
  },
  methods: {
    async signUpRequest(userData) {
      const APIURL = `${process.env.VUE_APP_API_URL}/users`;

      try {
        await this.$axios.post(APIURL, userData)
        alert('회원가입 완료!\n로그인 해주세요.')
        this.$router.push('/signin')
      } catch(err) {
        alert('이미 가입된 ID입니다.')
      }
    },
    async signUp() {

      if(!this.valid) {
        return;
      }

      if(!this.passwordConfirmMatcher) {
        alert('비밀번호 확인이 같지 않습니다.');
        return;
      }

      // FIXME : userType 하드코드 추후 수정
      const userData = {
        gabia_id: this.userId,
        name: this.nickName,
        password: this.password
      }     

      await this.signUpRequest(userData);
    }
  }
}
</script>