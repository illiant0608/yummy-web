<template>
    <div>
        <el-tabs v-model="activeName">
            <el-tab-pane label="个人资料" name="first"></el-tab-pane>
        </el-tabs>
        <el-form ref="form" :model="memberInfo" label-width="80px">
            <el-form-item label="用户名:">
                <el-input v-model="memberInfo.memberName" :disabled="this.nameChange" style="width:200px;float:left;"></el-input>
            </el-form-item>
            <el-form-item label="注册电话:">
                <el-input v-model="memberInfo.phoneNumber" :disabled="this.phoneNumberChange" style="width:200px;float:left;"></el-input>
            </el-form-item>


        </el-form>

        <el-row>
            <el-col :span="12">
                <div class="btns">
                    <el-button type="primary" @click="changeInfo">修改</el-button>
                    <el-button type="primary" @click="submitChange">提交</el-button>
                </div>
            </el-col>
            <el-col :span="12">
                <div class="writeoff_btn">
                    <el-button type="danger" @click="alertVisible = true">注销</el-button>
                </div>
            </el-col>
        </el-row>

        <el-tabs v-model="activeName1">
            <el-tab-pane label="地址管理" name="first"></el-tab-pane>
        </el-tabs>

        <div class="address_container">
            <el-row :gutter="20">
                <el-col :span="8" v-for="(item, index) in address" :key="item.addressId">
                    <AddressCard :addressInfo="item" @addressInfoChange="onInfoChange" @addressAdded="onAdd" @addressInfoDelete="onDelete"></AddressCard>
                </el-col>
            </el-row>

        </div>

        <el-dialog
                title="提示"
                :visible.sync="alertVisible"
                width="30%">
            <div class="writeoff_alert">
                <h2>您确定要注销账号吗？</h2>
                <el-button type="danger" @click="writeOff">确定</el-button>
            </div>
        </el-dialog>


    </div>
</template>

<script>
    function sleep(time) {
        return new Promise((resolve) => setTimeout(resolve, time));
    }

    import AddressCard from '../components/addressCard'
    import Vue from 'vue'
    export default {
        name: 'memberSettings',
        components: {
            AddressCard
        },
        data() {
            return {
                activeName: 'first',
                activeName1: 'first',
                memberInfo: {

                },
                nameChange: true,
                phoneNumberChange: true,
                address: [],
                emptyAddress: {
                    memberId: this.$route.params.id,
                    address: '',
                    contactName: '',
                    phoneNumber: ''
                },
                alertVisible: false

            }
        },
        methods: {
            onInfoChange(val) {
                for(var i = 0;i < this.address.length;i++) {
                    if (this.address[i].addressId == val.addressId) {
                        Vue.set(this.address, i, val)
                    }
                }
            },
            onAdd(val) {
                Vue.set(this.address, this.address.length-1, val)
                Vue.set(this.address, this.address.length, this.emptyAddress) //需要把空的地址重新加上去
            },
            onDelete(val) {

                for (var i = 0;i < this.address.length;i++) {
                    if (this.address[i].addressId == val) {
                        this.address.splice(i, 1)
                    }
                }

                let param = new URLSearchParams()
                param.append("addressId", val);
                this.axios.post('http://localhost:8080/deleteAddress', param).then(response => {
                    this.$message("删除成功")
                })
            },
            changeInfo() {
                this.nameChange = false;
                this.phoneNumberChange = false;
            },
            submitChange() {
                console.log(this.memberInfo)
                this.axios.post('http://localhost:8080/modifyInfo', this.memberInfo).then(response => {
                    this.nameChange = true
                    this.phoneNumberChange = true
                })
            },
            writeOff() {
                let param = new URLSearchParams()
                param.append("memberId", this.$route.params.id)
                this.axios.post('http://localhost:8080/writeOff', param).then(response => {
                    //console.log(response)
                    this.$message('注销成功，即将前往首页')
                    sleep(2000).then(() => {
                        this.$router.push('/')
                    })
                })
            }


        },

        mounted() {
            let param = new URLSearchParams()
            param.append("memberId", localStorage.getItem('ID'))
            this.axios.post('http://localhost:8080/getMemberInfo', param).then(response => {

                this.memberInfo = response.data.data
                this.address = response.data.data.addresses
                //加一个空的地址，为了最后显示一个add卡片

                this.address.push(this.emptyAddress)
            })
        }
    }
</script>

<style>
    .btns {
        float: left;
        margin-left: 80px;
    }

    .writeoff_btn {
        float: right;
        margin-right: 130px;
    }

    .writeoff_alert {
        margin-bottom:  30px;
    }

</style>