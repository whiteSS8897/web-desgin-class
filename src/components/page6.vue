<template>
    <div class="button save_button" @click="save_data">儲存搜尋紀錄</div>
    <div style="display:flex; align-items:start;">
        <div class="raid_power_area">
            <div class="calculator_title">戰地硬幣小算盤</div>
            <div style="background-color:#ffffbb66; padding:5px; text-align:center;">
                戰鬥力：<input type=text class="raid_power_input" v-model="raid_power_display">
            </div>
            <div style="text-align:center; margin-top:6px;">
                <div>每 {{ Round_to(100000000000/raid_power,2).toLocaleString('en-US',{maximumFractionDigits:20}) }} 秒獲得一個硬幣</div>
                <div>每天獲得 {{ Round_to(86400/100000000000*raid_power,2).toLocaleString('en-US',{maximumFractionDigits:20}) }} 個硬幣</div>
                <div>每周獲得 {{ Round_to(86400/100000000000*raid_power*7,2).toLocaleString('en-US',{maximumFractionDigits:20}) }} 個硬幣</div>
            </div>
        </div>

        <div class="attack_BossDamage_area">
            <div class="calculator_title">攻% or B傷效益小算盤</div>
            <div style="background-color:#ffffbb66; padding:5px; text-align:center;">
                目前B+總：<input type=number class="total_damage_input no_arrow" v-model="total_damage">%
                <div style="margin-top:8px;"></div>
                <input type=number class="total_damage_input no_arrow" v-model="damage">%B傷，
                <input type=number class="total_damage_input no_arrow" v-model="attack_p">%攻
            </div>
            <div style="text-align:center; margin-top:6px;">
                <div>當你的總攻%＞{{ ((total_damage/100*attack_p-damage+attack_p)/damage*100).toFixed(2) }}% 後,
                <br>
                {{ damage }}%B傷＞{{ attack_p }}%攻</div>
            </div>
        </div>
    </div>
    <div style="display:flex; align-items:start; margin-top:50px;">
        <div class="ignore_def_add_area">
            <div class="calculator_title"><span style="color:#ff0000;">多</span>無視小算盤</div>
            <div style="background-color:#ffffbb66; padding:5px; text-align:center;">
                原無視：<input type=number class="ignore_def_input no_arrow" v-model="ignore_def_1">%
                <div style="margin-top:8px;"></div>
                <span style="font-size:18px; color:#ff0000; font-weight:900;">多</span>
                <input type=number class="ignore_def_input no_arrow" v-model="ignore_def_add">%
            </div>
            <div style="text-align:center; font-size:20px;">⇒變 {{ Round_to(ignore_def_1+(100-ignore_def_1)*ignore_def_add/100,4) }}%</div>
        </div>
        <div class="ignore_def_minus_area">
            <div class="calculator_title"><span style="color:#ff0000;">少</span>無視小算盤</div>
            <div style="background-color:#ffffbb66; padding:5px; text-align:center;">
                原無視：<input type=number class="ignore_def_input no_arrow" v-model="ignore_def_2">%
                <div style="margin-top:8px;"></div>
                <span style="font-size:18px; color:#ff0000; font-weight:900;">少</span>
                <input type=number class="ignore_def_input no_arrow" v-model="ignore_def_minus">%
            </div>
            <div style="text-align:center; font-size:20px;">⇒變 {{ Round_to(100*(ignore_def_2-ignore_def_minus)/(100-ignore_def_minus),4) }}%</div>
        </div>
    </div>
</template>


<script setup>
import {computed, reactive, ref, watch, onBeforeMount, onMounted} from "vue";

const Sum_of_Nums = (arr)=>{
    if(arr.length === 0){return 0;}
    return arr.reduce((a,b)=>a+b);
}

const raid_power = ref(0);
const raid_power_display = ref(0);
watch(raid_power_display,(new_num)=>{
    let arr = new_num.split("");
    let temp = new Array();
    let temp_flag = (arr.indexOf(".") === arr.length-1 && arr.length !== 0);
    for(let c of arr){if(!isNaN(c) || c==="."){temp.push(c);}}
    if(isNaN(Number(temp.join("")))){temp.pop();}
    temp = Number(temp.join(""));
    raid_power.value = temp;
    temp = temp.toLocaleString('en-US',{maximumFractionDigits:20});
    temp = temp.toString();
    if(temp_flag){temp += ".";}
    raid_power_display.value = temp;
})
const Round_to = (num,f)=>{return Math.round(num*10**f)/10**f;}

const total_damage = ref(0);
const damage = ref(0);
const attack_p = ref(0);

const ignore_def_1 = ref(0);
const ignore_def_add = ref(0);
const ignore_def_2 = ref(0);
const ignore_def_minus = ref(0);


import data from "/src/assets/SettingConfig.json";
var _settings = reactive({});
var _frontend = reactive({});
onMounted(()=>{
    _settings = data;
    _frontend = _settings["FrontEnd"]
    get_data();
});


const props = defineProps({
    user_name:{},
})
const _now_user = computed(()=>{
        return props.user_name
    })
const save_data = async ()=>{
    const calculatorData = {
                    "Calculator1":{"Combat Effectiveness":raid_power.value},
                    "Calculator2":{"Current Total of Boss Damage":total_damage.value,
                                    "Boss Damage Efficiency":damage.value,
                                    "Attack Damage Efficiency":attack_p.value},
                    "Calculator3":{"Original":ignore_def_1.value,
                                    "Increase":ignore_def_add.value},
                    "Calculator4":{"Original":ignore_def_2.value,
                                    "Decrease":ignore_def_minus.value}}
    const _body = {
        "userName": _now_user.value,
        "calculatorData": calculatorData
    }
    const requestOptions = {
                            method:"POST",
                            headers:{
                                "Content-Type": "application/json"
                            },
                            body:JSON.stringify(_body)
    }
    console.log(JSON.stringify(_body))
    await fetch("http://"+_frontend["Hostname"]+":"+_frontend["Backend_port"]+"/saveCalculatorData/",requestOptions)
    .then(res =>{
        return res.json();
    })
    .then(res =>{
        console.log(res);
    })
    .catch(res =>{
        console.log(res);
    })
}

const get_data = async ()=>{
    const _body = {
        "userName": _now_user.value
    }
    const requestOptions = {
                            method:"POST",
                            headers:{
                                "Content-Type": "application/json"
                            },
                            body:JSON.stringify(_body)
    }
    await fetch("http://"+_frontend["Hostname"]+":"+_frontend["Backend_port"]+"/getCalculatorData/",requestOptions)
    .then(res =>{
        return res.json();
    })
    .then(res =>{
        return JSON.parse(res["calculatorData"]);
    })
    .then(res =>{
        console.log(res);
        raid_power.value = res["Calculator1"]["Combat Effectiveness"];
        raid_power_display.value = raid_power.value.toString();
        total_damage.value = res["Calculator2"]["Current Total of Boss Damage"];
        damage.value = res["Calculator2"]["Boss Damage Efficiency"];
        attack_p.value = res["Calculator2"]["Attack Damage Efficiency"];
        ignore_def_1.value = res["Calculator3"]["Original"];
        ignore_def_add.value = res["Calculator3"]["Increase"];
        ignore_def_2.value = res["Calculator4"]["Original"];
        ignore_def_minus.value = res["Calculator4"]["Decrease"];
    })
    .catch(res =>{
        console.log(res);
    })
}
</script>


<style scoped>
input{
    background-color:#ffffff;
    border:#888888 solid 1px;
    border-radius:6px;
    text-align:center;
}
.calculator_title{
    background-color:#ffff88;
    text-align:center;
    padding:8px;
    font-size:18px;
    font-weight:900;
}
.raid_power_input{
    width:200px;
    font-size:18px;
}
.raid_power_area{
    width:300px;
    border:#ffff88 dashed 3px;
}
.attack_BossDamage_area{
    width:300px;
    border:#ffff88 dashed 3px;
    margin-left:40px;
}
.total_damage_input{
    width:70px;
    font-size:18px;
}
.no_arrow::-webkit-inner-spin-button ,.no_arrow::-webkit-outer-spin-button{
    -webkit-appearance: none;
    -moz-appearance: none;
    appearance: none;
}
.ignore_def_input{
    width:100px;
    font-size:18px;
}
.ignore_def_add_area{
    width:250px;
    border:#ffff88 dashed 3px;
}
.ignore_def_minus_area{
    width:250px;
    border:#ffff88 dashed 3px;
    margin-left:10px;
}
.save_button{
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
    margin-bottom:24px;
    width:fit-content;
}
.save_button:hover{
    background-color:#444444;
}
</style>
