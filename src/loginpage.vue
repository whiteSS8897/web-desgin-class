<template>
    <div v-if="!sign_in_flag">
        <div style="height:100vh; display:flex; align-items:center; justify-content:center;">
            <div id="login_page" class="login_page">
                <div style="display:flex; justify-content:center; align-items:center; font-size:1.5rem; margin-top:30px;">
                    <img src="/src/assets/imgs/maplestory.png" style="width:2rem;">
                    楓谷資料站登入介面
                </div>
                <form style="margin-top:30px;">
                    <fieldset style="border:none; width:calc(100% - 38px); padding:15px;">
                        <div style="margin-left:15px;">
                            <div style="font-size:1.2rem;">&#10146;帳號</div>
                            <input id="account" name="loginpage" placeholder="請輸入帳號" class="log_input" v-model="_account" @keyup.enter.prevent="_log_in"/>
                            <br/>
                            <div style="height:20px;"></div>
                            
                            <div style="font-size:1.2rem;">&#10146;密碼</div>
                            <input id="password" name="loginpage" type="password" placeholder="請輸入密碼" class="log_input" v-model="_password" @keyup.enter.prevent="_log_in"/>
                            <br/>
                            <div style="margin-bottom:0px;"></div>
                            
                            <input id="show_password" name="loginpage" type="checkbox" v-model="show_password"/>
                            <label for="show_password">顯示密碼</label>
                            <br/>
                            <div style="height:60px;"><div v-if="wrong_login" style="padding:10px 0 0 10px; color:#dd0000;">※錯誤的帳號或密碼</div></div>
                        </div>
                    </fieldset>
                </form>
                <div style="display:flex; align-items:center;">
                    <div class="button login_button" @click="_log_in">登入</div>
                    <div class="button sign_in_button" @click="open_sign_in_page">註冊帳號</div>
                </div>
                <!-- <div class="button login_button" @click="_check">檢查</div> -->
                <!-- <div>{{ _token }}</div> -->
            </div>
            <!-- <div class="button login_button" @click="_href">連結</div> -->
        </div>
    </div>
    <sign_in_page v-if="sign_in_flag" @open_log_in="open_log_in_page"/>
</template>

<script setup>
import {computed, reactive, ref, onMounted, watch} from "vue";

import data from "/src/assets/SettingConfig.json";
    var _settings = reactive({});
    var _frontend = reactive({});
    onMounted(()=>{
        _settings = data;
        _frontend = _settings["FrontEnd"]
    });
    // const _href= ()=>{document.location.href="http://localhost:5173/";}
    
    const show_password = ref(false);
    watch(show_password,(ifshow)=>{
        let p = document.querySelector("#password");
        if(ifshow){p.type = "text";}
        else{p.type = "password";}
    })


    
    const _account = ref('');
    const _password = ref('');
    const _token = ref('');
    const now_user = ref("unknown");
    const now_nickname = ref("unknown");

    
    const wrong_login = ref(false)


    import cookies from "vue-cookies"
    const fake_cookie = ref();
    const _log_in = async () =>{
        const _body = {"password":_password.value,"username":_account.value}
        const requestOptions = {
                                method:"POST",
                                headers:{
                                    "Content-Type": "application/json"
                                },
                                body:JSON.stringify(_body)

        }
        await fetch("http://"+_frontend["Hostname"]+":"+_frontend["Backend_port"]+"/login/",requestOptions)
        .then(res =>{
            return res.json();
        })
        .then(res =>{
            console.log(res);
            _token.value = res["token"]
            console.log(_token.value)
            // cookies.set("token",_token.value)
            fake_cookie.value = _token.value
            emit("fake_cookie_emit",fake_cookie.value)
            now_user.value = _account.value;
            _check()
        })
        .catch(err =>{
            console.log(err);
        })
    };

    const props = defineProps({
        _kli:{
            type:Boolean
        }
    })

    const kli = computed(()=>{
        return props._kli
    })

    const now_kli = ref(kli.value)

    const emit = defineEmits(["check_login","get_login_user","get_login_nickname","fake_cookie_emit"])
    watch(now_kli,(newkli)=>{
        emit("check_login",newkli)
        //console.log("change:",newkli)
    })

    const _check = async () =>{
        // let check_token = cookies.get("token")
        let check_token = fake_cookie.value
        console.log("checking token：",check_token)
        const _body = {"token":check_token}
        const requestOptions = {
                                method:"POST",
                                headers:{
                                    "Content-Type": "application/json"
                                },
                                body:JSON.stringify(_body)
        }
        await fetch("http://"+_frontend["Hostname"]+":"+_frontend["Backend_port"]+"/checkToken/",requestOptions)
        .then(res =>{
            return res.json();
        })
        .then(res =>{
            console.log(res);
            const login_page = document.querySelector("#login_page");
            if (res["message"] === "Token is valid"){
                //console.log(now_kli.value)
                now_kli.value = true
                emit("get_login_user",now_user.value)

                now_nickname.value = res["nickname"]
                emit("get_login_nickname",now_nickname.value)
                console.log("username:",now_user.value)
                console.log("nickname:",now_nickname.value)
                //console.log(now_kli.value)
            }
            else{
                //console.log(now_kli.value)
                now_kli.value = false
                //console.log(now_kli.value)
                login_page.classList.add("wrong_shake");
                wrong_login.value = true;
                setTimeout(() =>{login_page.classList.remove("wrong_shake");},450)
            }
        })
    }

import sign_in_page from "./signpage.vue"
const sign_in_flag = ref(false)
const open_sign_in_page = ()=>{sign_in_flag.value = true;}
const open_log_in_page = ()=>{sign_in_flag.value = false;}
</script>

<style scoped>
.login_page{
    background-color:#bbbbbbee;
    width:400px;
    height:50%;
    overflow:auto;
    border-radius:10px;
    box-shadow:0 0 30px 10px #ffffff88,0 0 20px 20px #ffffff88 inset;
}
.button{  /* 按鈕(所有應該要可以點擊的東西) */
    cursor:pointer;  /* 滑鼠會變點擊圖樣(手指) */
    transition-duration:0.2s;  /* 因為按鈕都有做滑鼠移入會變色 */
}
.login_button{
    background-color:#0066ff;
    color:#ffffff;
    width:70px;
    padding-top:6px;
    padding-bottom:6px;
    margin-left:32px;
    border-radius:10px;
    display:flex;
    justify-content:center;
    align-items:center;
    box-shadow: 2px -2px 4px 1px #00000066 inset,-1px 1px 4px 1px #ffffff88 inset;
}
.login_button:hover{
    background-color:#0000dd;
}
.log_input{
    font-size:1rem;
    width:100%;
    padding:10px;
    background-color:#ffffff;
    border-radius:6px;
}
@keyframes shake{
    0%{
        margin-left:0;
    }25%{
        margin-left:15px;
    }50%{
        margin-left:0;
    }75%{
        margin-left:-15px;
    }100%{
        margin-left:0;
    }
}
.wrong_shake{
    animation:shake .15s ease-in-out 3;
}
.sign_in_button{
    font-size:16px;
    background-color:#777777;
    color:#ffffff;
    padding:4px 10px;
    font-size:14px;
    border-radius:10px;
    display:flex;
    justify-content:center;
    align-items:center;
    box-shadow: 2px -2px 4px 1px #00000066 inset,-1px 1px 4px 1px #ffffff88 inset;
    margin-left:16px;
}
.sign_in_button:hover{
    background-color:#444444;
}
</style>
