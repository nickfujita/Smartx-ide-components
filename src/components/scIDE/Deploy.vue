<template>
  <div class="deploy-page">

    <div class="deploy-card">
      <button class="btn btn-outline-success deploy-btn-submit deploy-btn-submit-wallet" data-toggle="modal" data-target="#WalletFileInfoInDeploy">{{$t('deploy.selectWallet')}}</button>
      <button class="btn btn-outline-success deploy-btn-submit deploy-btn-submit-wallet" style="float: right" data-toggle="modal" data-target="#GenerateWallet">{{$t('deploy.generateWallet')}}</button>

    </div>
    <div class="deploy-card card-info" >
      <div class="card border-secondary mb-3" style="max-width: 20rem;">
        <div class="card-header">{{$t('deploy.walletInfo')}}</div>
        <div class="deploy-card-scroll">
          <div class="card-body">
            <p class="card-text test-title-text test-card-text-title" style="margin-top: 0px"><strong>{{ $t('test.selectNet') }}</strong></p>
            <label class="card-text test-title-text"><input name="deployNet" type="radio" v-model="network" value="0" @change="getNetworkAsset()"/><strong style="margin-left: 4px">{{ $t('test.mainNet') }}</strong></label>
            <label class="card-text test-title-text" style="margin-left: 8px"><input name="deployNet" type="radio" v-model="network" value="1"  @change="getNetworkAsset()"/><strong style="margin-left: 4px">{{ $t('test.testNet') }}</strong></label>
            <label class="card-text test-title-text" style="margin-left: 8px"><input name="deployNet" type="radio" v-model="network" value="2"  @change="getNetworkAsset()"/><strong style="margin-left: 4px">{{ $t('test.privateNet') }}</strong></label>
            <input class="test-private-net-input" v-show="network === '2'&& !isHidePrivateNetInput" v-model="privateNet" >
            <button v-show="network === '2' && !isHidePrivateNetInput" @click="privateNetInputState">ok</button>
            <a v-show="network === '2' && isHidePrivateNetInput">{{privateNet}}</a>
            <button v-show="network === '2' && isHidePrivateNetInput" @click="privateNetInputState">Cancel</button>
          </div>
          <div  v-show="showWalletInfo" class="card-body">
            <span class="card-text"><strong>{{ $t('deploy.address') }}</strong></span>
            <span class="card-text">{{ deployWalletInfo.info.address }}</span>
          </div>
          <div  v-show="showWalletInfo" class="card-body">
            <span class="card-text"><strong>ONT:</strong></span>
            <span class="card-text">{{deployWalletInfo.info.ont}} </span>
          </div>
          <div v-show="showWalletInfo" class="card-body card-last-body">
            <span class="card-text"><strong>ONG:</strong></span>
            <span class="card-text">{{deployWalletInfo.info.ong}}</span>
          </div>
        </div>
      </div>
    </div>

    <div class="deploy-card card-fee" >
      <div class="card border-secondary mb-3" style="max-width: 20rem;">
        <div class="card-header" data-toggle="tooltip" data-placement="bottom" :title="$t('deploy.infoTooltips')">{{$t('deploy.info')}}</div>
        <div class="deploy-card-scroll">
          <div class="card-body">
            <label class="card-text deploy-info-card-text">{{ $t('deploy.name') }}</label>
            <input class="deploy-input" v-model="deployContractInfo.name">
          </div>
          <div class="card-body">
            <label class="card-text deploy-info-card-text">{{ $t('deploy.version') }}</label>
            <input class="deploy-input" v-model="deployContractInfo.version">
          </div>
          <div class="card-body">
            <label class="card-text deploy-info-card-text">{{ $t('deploy.author') }}</label>
            <input class="deploy-input" v-model="deployContractInfo.author">
          </div>
          <div class="card-body">
            <label class="card-text deploy-info-card-text">{{ $t('deploy.email') }}</label>
            <input class="deploy-input" v-model="deployContractInfo.email">
          </div>
          <div class="card-body">
            <label class="card-text deploy-info-card-text card-last-body">{{ $t('deploy.desc') }}</label>
            <input class="deploy-input" v-model="deployContractInfo.desc">
          </div>
        </div>
      </div>
    </div>

    <div class="deploy-card card-result" >
      <div class="card border-secondary mb-3" style="gitmax-width: 20rem;">
        <div class="card-header" data-toggle="tooltip" data-placement="bottom" :title="$t('deploy.resultTooltips')">{{$t('deploy.result')}}</div>
        <div class="deploy-card-scroll">
          <div class="card-body deploy-card-scroll" v-if="deployInfo.info.result" >
            <p class="card-text-center card-text">{{deployInfo.info.result}}</p>
            <br/>
            <br/>
          </div>
          <div class="card-body" v-else>
            <p class="card-text card-text-center">{{this.deployStatus ? $t('deploy.deployed') : $t('deploy.noData')}}</p>
          </div>
        </div>
      </div>
    </div>

    <div class="deploy-card">
      <!--<button class="btn btn-outline-success deploy-btn-submit" v-bind:disabled="waitingStatus" @click="doDeploy">{{waitingStatus ? $t('deploy.waiting') : $t('deploy.deploy')}}</button>-->
      <button class="btn btn-outline-success deploy-btn-submit" data-toggle="modal" v-bind:disabled="waitingStatus" @click="showEnterWalletPassword" >{{waitingStatus ? $t('deploy.waiting') : $t('deploy.deploy')}}</button>
    </div>

    <!--Wallet Modal -->
    <div class="modal fade devlop-modal" id="WalletFileInfoInDeploy" tabindex="-1" role="dialog" >
      <div class="modal-dialog" role="document">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title" id="exampleModalLabel">{{$t('deploy.selectWallet')}}</h5>
            <button type="button" class="close" data-dismiss="modal" aria-label="Close">
              <span aria-hidden="true">&times;</span>
            </button>
          </div>
          <div class="modal-body">
            <div class="form-group">
              <div class="custom-file devlop-custom-file">
                <input type="file" @change="onFileChange" class="deploy-custom-file-input" id="exampleInputFileInDeploy" aria-describedby="fileHelp"
                       name="file" v-validate data-vv-rules="required">
                <label id="deploy-input-file-label" class="deploy-custom-file-label" for="exampleInputFileInDeploy" >{{FileName}}</label>
              </div>
              <small class="form-text text-muted deploy-err-message" v-show="errors.has('file')">{{ errors.first('file') }}</small>
            </div>
            <div class="form-group">
              <div class="input-group">
                <input :type="[isShowPassword ? 'text' : 'password']"
                       v-model="password"
                       v-validate data-vv-rules="required|min:6"
                       class="form-control deploy-input" name="password" :placeholder="$t('deploy.enterPw')">
                <div class="input-group-append deploy-input-group-append" @click="viewPassword">
                    <span class="input-group-text">
                      <i class="fa" :class="[isShowPassword ? 'fa-eye' : 'fa-eye-slash']" aria-hidden="true"></i>
                    </span>
                </div>
              </div>
              <small class="form-text text-muted deploy-err-message" v-show="errors.has('password')">{{ errors.first('password') }}</small>
            </div>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary deploy-dialog-btn-close" data-dismiss="modal">{{$t('deploy.close')}}</button>
            <button type="button" class="btn btn-primary deploy-dialog-btn" v-bind:disabled="waitingUnlockWallet" :data-dismiss="[closeDialog ? 'modal' : '']" @click="unlockWalletFile">{{waitingUnlockWallet ? $t('deploy.waitingUnlock') : $t('deploy.unlock')}}</button>
          </div>
        </div>
      </div>
    </div>

    <!--Enter Wallet Password Modal -->
    <div class="modal fade devlop-modal" id="enterWalletPassword" tabindex="-1" role="dialog" >
      <div class="modal-dialog" role="document">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title" >{{$t('deploy.selectWallet')}}</h5>
            <button type="button" class="close" data-dismiss="modal" aria-label="Close">
              <span aria-hidden="true">&times;</span>
            </button>
          </div>
          <div class="modal-body">
            <div class="form-group">
              <div class="input-group">
                <input :type="[isShowPassword ? 'text' : 'password']"
                       v-model="getWalletPrivateKeyPassowrd"
                       class="form-control deploy-input" name="password" :placeholder="$t('deploy.enterPw')">
                <div class="input-group-append deploy-input-group-append" @click="viewPassword">
                    <span class="input-group-text">
                      <i class="fa" :class="[isShowPassword ? 'fa-eye' : 'fa-eye-slash']" aria-hidden="true"></i>
                    </span>
                </div>
              </div>
              <small class="form-text text-muted deploy-err-message" v-show="errors.has('password')">{{ errors.first('password') }}</small>
            </div>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary deploy-dialog-btn-close" data-dismiss="modal">{{$t('deploy.close')}}</button>
            <button type="button" class="btn btn-primary deploy-dialog-btn" v-bind:disabled="waitingUnlockWallet" :data-dismiss="[closeDialog ? 'modal' : '']" @click="getWalletPrivateKey">{{waitingUnlockWallet ? $t('deploy.waitingUnlock') : $t('deploy.unlock')}}</button>
          </div>
        </div>
      </div>
    </div>

    <!--Generate Wallet -->
    <div class="modal fade devlop-modal" id="GenerateWallet" tabindex="-1" role="dialog" >
      <div class="modal-dialog" role="document">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title" >{{$t('deploy.generateWallet')}}</h5>
            <button type="button" class="close" data-dismiss="modal" aria-label="Close">
              <span aria-hidden="true">&times;</span>
            </button>
          </div>
          <div class="modal-body">
            <div class="form-group">
              <div class="input-group">
                <input :type="[isShowPassword ? 'text' : 'password']"
                       v-model="generateWalletPassword"
                       class="form-control deploy-input" name="password" :placeholder="$t('deploy.enterPw')">
                <div class="input-group-append deploy-input-group-append" @click="viewPassword">
                    <span class="input-group-text">
                      <i class="fa" :class="[isShowPassword ? 'fa-eye' : 'fa-eye-slash']" aria-hidden="true"></i>
                    </span>
                </div>
              </div>
              <small class="form-text text-muted deploy-err-message" v-show="errors.has('password')">{{ errors.first('password') }}</small>
            </div>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary deploy-dialog-btn-close" data-dismiss="modal">{{$t('deploy.close')}}</button>
            <button type="button" class="btn btn-primary deploy-dialog-btn" v-bind:disabled="waitingUnlockWallet" :data-dismiss="[closeDialog ? 'modal' : '']" @click="generateWallet">{{waitingUnlockWallet ? $t('deploy.waitingGenerate') : $t('deploy.generate')}}</button>
          </div>
        </div>
      </div>
    </div>

    <!-- Error Modal -->
    <div class="modal fade devlop-modal" id="DeployError" tabindex="-1" role="dialog" >
      <div class="modal-dialog" role="document">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title" id="">{{$t('deploy.errorTitle')}}</h5>
            <button type="button" class="close" data-dismiss="modal" aria-label="Close">
              <span aria-hidden="true">&times;</span>
            </button>
          </div>
          <div class="modal-body">
              <label class="error-modal-body-text">{{ErrorInfo}}</label>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary deploy-dialog-btn-close" data-dismiss="modal">{{$t('deploy.close')}}</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import {mapState} from 'vuex'
  import zh from './../../common/lang/zh'
  import en from './../../common/lang/en'
  import LangStorage from './../../helpers/lang'
  import Sleep from './../../helpers/sleep'
  import FileHelper from './../../common/ont-wallet/file-generate-and-get'
  import OWallet from './../../common/ont-wallet/wallet'
  let Ont = require('ontology-ts-sdk');


  export default {
    name: "deploy",
    data() {
      return {
        waitingStatus: false,
        deployStatus : false,
        ErrorInfo: '',
        showWalletInfo: false,
        password: '',
        isShowPassword: false,
        FileName: '',
        WalletFile: '',
        closeDialog : false,
        waitingUnlockWallet: false,
        network:'1',
        privateNet:'http://127.0.0.1:20334',
        isHidePrivateNetInput:false,
        getWalletPrivateKeyPassowrd:'',
        generateWalletPassword:''
      }
    },
    created(){
    },
    computed: {
      /*
        projectInfo:{
          info:{
            abi:'',
            code:'',
            contract_hash:'',
            created_at:'',
            id:'',
            info_author:'',
            info_desc:'',
            info_email:'',
            info_name:'',
            info_version:'',
            language:'',
            name:'',
            nvm_byte_code:'',
            type:'',
            updated_at:'',
            user_id:'',
            wat:''
        }，
        projectName:{
          info:{
            id:'',
            language:'',
            projectName:'',
          }
        }
        deployContractInfo:{
          author:'',
          desc:'',
          email:'',
          name:'',
          version:'',
        }
        deployInfo:{
          info:{
            result:{
              Action:'',
              Desc:'',
              Error:'',
              Result:{
                GasConsumed:'',
                Notify:[{
                  ContractAddress:'',
                  States:[{
                    0:'',//Transfer type
                    1:'',//From,
                    2:'',//To
                    3:'',//GasConsumed
                  }]
                }]
                State:'',
                TxHash:''
              },
              Version:''
            }
          }
        }
        compileInfo:{
          abi{
            function:[{
              name:'',
              parameters:[{
                name:'',
                type:''
              }]
              returntype:''
            }],
            avm:'',
            contractHash:'',
            errdetail:'',
            haveReCompile:'',
            showCompileInfo: '',
          }
        }
       */
      ...mapState({
        projectInfo: state => state.ProjectInfoPage.ProjectInfo,
        projectName: state => state.ProjectInfoPage.ProjectName,
        deployWalletInfo: state => state.DeployPage.DeployWalletInfo,
        deployContractInfo: state => state.DeployPage.DeployContractInfo,
        deployInfo: state => state.DeployPage.DeployInfo,
        compileInfo : state => state.CompilePage.CompileInfo
      })
    },
    mounted(){
    },
    watch: {
    },
    methods:{
      privateNetInputState(){
        this.isHidePrivateNetInput = !this.isHidePrivateNetInput
        if(this.isHidePrivateNetInput){
          if(this.deployWalletInfo.info){
            this.getNetworkWalletInfo(this.deployWalletInfo.info,this)
          }
        }else{

        }
      },
      onFileChange() {
        let files = document.getElementById("exampleInputFileInDeploy").files
        if (!files.length){
          this.FileName = (LangStorage.getLang('zh') === "zh") ? zh.login.chooseFile : en.login.chooseFile
          return
        }
        this.FileName = files[0].name
        this.WalletFile = files[0]
      },
      unlockWalletFile(){
        let _self = this
        this.$validator.validateAll().then(result => {
          if (result) {
            this.waitingUnlockWallet = true
            Sleep.sleep(200).then(() => {
              FileHelper.readWalletFile(this.WalletFile).then((walletFile) => {
                if(walletFile) {
                  let account = OWallet.decryptWalletFile(JSON.parse(walletFile), this.password)
                  if(account ) {
                    _self.getNetworkWalletInfo(account,_self)
                    _self.showWalletInfo = true
                    $("#WalletFileInfoInDeploy").modal("hide");
                  }else{
                    let title = (LangStorage.getLang('zh') === "zh") ? zh.deploy.errorTitle : en.deploy.errorTitle
                    let content = (LangStorage.getLang('zh') === "zh") ? zh.deploy.errowWalletPassword : en.deploy.errowWalletPassword
                    this.showErrorModel(title,content,true)
                    $("#WalletFileInfoInDeploy").modal("hide");
                  }
                }else{
                }
              })
              this.waitingUnlockWallet = false
            })
          }else{
          }
        })
      },
      getNetworkAsset(){
        if(this.deployWalletInfo.info){
          this.getNetworkWalletInfo(this.deployWalletInfo.info.account,this)
        }
      },
      getNetworkWalletInfo($account,_self){
        let defaultNet
        if(this.network === '0'){
          defaultNet = process.env.NODE_URL
        }else if(this.network === '1'){
          defaultNet = "https://polaris1.ont.io:10334"
        }else{
          defaultNet = this.privateNet
        }
        let payload = {
          network: defaultNet,
          account: $account
        }
        _self.$store.dispatch('setDeployWallet', payload).then(result => {

        })
      },
      showEnterWalletPassword(){
        if(this.deployWalletInfo.info) {
          $("#enterWalletPassword").modal("show");
        }else{
          let title = (LangStorage.getLang('zh') === "zh") ? zh.deploy.errorTitle : en.deploy.errorTitle
          let content = (LangStorage.getLang('zh') === "zh") ? zh.deploy.errorWallet : en.deploy.errorWallet
          this.showErrorModel(title,content,true)
        }
      },
      getWalletPrivateKey(){
        this.waitingUnlockWallet = true
        let account = this.deployWalletInfo.info.account
        let privateKey = OWallet.getAccountPrivateKey(account,this.getWalletPrivateKeyPassowrd)

        if(privateKey){
          this.waitingUnlockWallet = false
          $("#enterWalletPassword").modal("hide");
          this.doDeploy(privateKey)
        }else{
          this.waitingUnlockWallet = false
          let title = (LangStorage.getLang('zh') === "zh") ? zh.deploy.errorTitle : en.deploy.errorTitle
          let content = (LangStorage.getLang('zh') === "zh") ? zh.deploy.errowWalletPassword : en.deploy.errowWalletPassword
          this.showErrorModel(title,content,true)
          $("#enterWalletPassword").modal("hide");
        }

      },
      generateWallet(){
        let privateKey = Ont.Crypto.PrivateKey.random()
        const wif = privateKey.serializeWIF();
        let body = {
          label: this.label,
          privateKey: privateKey,
          password: this.generateWalletPassword,
          wif: wif
        }
        let account = OWallet.createJsonWalletWithPrivateKey(body)
        $("#GenerateWallet").modal("hide");

      },
      viewPassword() {
        this.isShowPassword = !this.isShowPassword
      },
      showRun(){
        this.$router.push({ name:'IDE-Run',params:{projectName:this.projectName.info.projectName}})
      },
      getContractHash(vmType){
        //will export a method in sdk
        let contractAddr = Ont.Crypto.Address.fromVmCode(this.compileInfo.avm);
        const hash = contractAddr.serialize();
        const codeHash = contractAddr.toHexString();
        // let contractHash = Ont.utils.num2hexstring(vmType) + hash.substr(2)
        this.$store.dispatch('setContractHash', codeHash)
        return codeHash
      },
     doDeploy($privateKey){
        if(!this.deployContractInfo.name || !this.deployContractInfo.version || !this.deployContractInfo.author ||
        !this.deployContractInfo.email || !this.deployContractInfo.desc) {
          //Need to be required for now.Will remove it when update on ontology-dapi
          this.ErrorInfo = (LangStorage.getLang('zh') === "zh") ? zh.deploy.emptyInfo : en.deploy.emptyInfo
          $('#DeployError').modal('show')
          this.waitingStatus = false
          return;
        }
        this.waitingStatus = true
        if(!this.compileInfo.avm) {
          this.ErrorInfo = (LangStorage.getLang('zh') === "zh") ? zh.deploy.errorCompile : en.deploy.errorCompile
          $('#DeployError').modal('show')
          this.waitingStatus = false
          return
        }

        //build deploycode tx
        const needStorage = true
        let avmCode = this.compileInfo.avm
        const name = this.deployContractInfo.name || ''
        const version = this.deployContractInfo.version || ''
        const author = this.deployContractInfo.author || ''
        const email = this.deployContractInfo.email || ''
        const desc = this.deployContractInfo.desc || ''

        let _self = this
       let account = this.deployWalletInfo.info.account


         let defaultNet
         if(this.network === '0'){
           defaultNet = process.env.NODE_URL
         }else if(this.network === '1'){
           defaultNet = "https://polaris1.ont.io:10334"
         }else{
           defaultNet = this.privateNet
         }
         const restClient = new Ont.RestClient(defaultNet);

         const tx = Ont.TransactionBuilder.makeDeployCodeTransaction(avmCode,name, version, author, email, desc, needStorage, '500', '30000000');
         tx.payer = new Ont.Crypto.Address(account.address);
         Ont.TransactionBuilder.signTransaction(tx, $privateKey);
         const result = restClient.sendRawTransaction(tx.serialize());

         result.then(function(value){

           _self.$store.dispatch('setDeployInfo', value)

           if(value.Desc === "SUCCESS"){
             _self.deployStatus = true
             _self.showRun()
             _self.waitingStatus = false

             let contractHash = _self.getContractHash()
             //save code to server
             let param = {
               id: _self.projectName.info.id,
               contract_hash: contractHash,
               info_name: _self.deployContractInfo.name,
               info_version: _self.deployContractInfo.version,
               info_author: _self.deployContractInfo.author,
               info_email: _self.deployContractInfo.email,
               info_desc: _self.deployContractInfo.desc,
             }
             _self.$store.dispatch('saveProject', param)
           }else{
             _self.deployStatus = true
             _self.waitingStatus = false
             _self.$store.dispatch('clearContractHash')
           }
         })






/*        this.$store.dispatch('getDapiProvider').then(provider => {
          if(!provider) {
            this.ErrorInfo = (LangStorage.getLang('zh') === "zh") ? zh.ide.noProvider : en.ide.noProvider
            $('#DeployError').modal('show')
            this.deployStatus = false
            this.waitingStatus = false
            return;
          }
          this.$store.dispatch('dapiDeploy', params).then(res => {
            //console.log(res)
            if(res === 'NO_ACCOUNT') {
              this.deployStatus = false
              this.waitingStatus = false
              this.ErrorInfo = (LangStorage.getLang('zh') === "zh") ? zh.ide.noProviderAccount : en.ide.noProviderAccount
              $('#DeployError').modal('show')
              return;
            }
            if(res === 'CANCELED') {
              this.deployStatus = false
              this.waitingStatus = false
              this.ErrorInfo = (LangStorage.getLang('zh') === "zh") ? zh.ide.deployFalseInProvider : en.ide.deployFalseInProvider
              $('#DeployError').modal('show')
              return;
            }
            let contractHash = this.getContractHash()
            this.$store.dispatch('setDeployInfo', res)
            this.deployStatus = true
            this.showRun()
            this.waitingStatus = false

            //save code to server
            let param = {
              id: _self.projectName.info.id,
              contract_hash: contractHash,
              info_name: _self.deployContractInfo.name,
              info_version: _self.deployContractInfo.version,
              info_author: _self.deployContractInfo.author,
              info_email: _self.deployContractInfo.email,
              info_desc: _self.deployContractInfo.desc,
            }
            this.$store.dispatch('saveProject', param)
            //console.log(res)
          })
        })*/
      },
      showErrorModel($title,$content,$isShowCloseButton){
        let payload = {
          title:$title,
          content:$content,
          isShowCloseButton:$isShowCloseButton
        }
        this.$store.dispatch('showLoadingModals',payload)
      }
    }
  }
</script>

<style scoped>
  .deploy-page{
    margin-left: 5px ;
    margin-right: 5px;
    height: 100%;
  }
  .deploy-btn-submit {
    border-radius: 0;
    width: 100%;
    color: white;
    border-color: #36a3bc;
    background-color: #36a3bc;
    font-size: 10px;
    padding: 8px 17.6px;
  }
  .deploy-btn-submit:hover,
  .deploy-btn-submit:active {
    background-color: #36a3bc;
    color: white;
  }
  .deploy-card{
    margin-top: 5px;
  }
  .card{
    max-width: 100% !important;
    height: 100% !important;
    border-radius:0;
  }
  .card-header{
    background-color: white;
    text-align: center;
    color: #36a3bc;
    padding-top: 6px !important;
    padding-bottom: 6px !important;
  }
  .card-body {
    padding-top: 15px !important;
    padding-bottom: 0 !important;
  }
  .card-last-body {
    padding-bottom: 15px !important;
  }
  .card-text{
    margin-bottom: 2px;
  }
  .card-text-center{
    text-align: center;
  }
  .card-fee{
    height: 45%;
  }
  .card-result{
    height: 30%;
    margin-bottom: -26px;
    padding-bottom: 26px;
  }

  .deploy-info-card-text {
    width: 70px;
    text-align: right;
  }

  .deploy-input{
    width: 70%;
    border-radius:0px;
  }
  .border-secondary{
    border-color: #C4C3C3 !important;
  }

  .deploy-dialog-btn-close{
    border-radius:0px;
  }
  .deploy-card-scroll{
    overflow-y:auto;
  }
  .devlop-modal{
    margin-top: 10%;
  }
  .error-modal-body-text{
    font-size: 16px;
  }

  .deploy-page{
    margin-left: 5px ;
    margin-right: 5px;
    height: 100%;
  }
  .deploy-card{
    margin-top: 5px;
  }
  .card{
    max-width: 100% !important;
    height: 100% !important;
    border-radius:0;
  }
  .card-header{
    background-color: white;
    text-align: center;
    color: #36a3bc;
    padding-top: 6px !important;
    padding-bottom: 6px !important;
  }
  .card-body {
    padding-top: 15px !important;
    padding-bottom: 0 !important;
  }
  .card-last-body {
    padding-bottom: 15px !important;
  }
  .card-text{
    margin-bottom: 2px;
  }
  .card-text-center{
    text-align: center;
  }

  .card-info{
    height: 35%;
    margin-top: -65px;
    padding-top: 69px;
  }
  .card-fee{
    height: 45%;
  }
  .card-result{
    height: 20%;
    margin-bottom: -26px;
    padding-bottom: 26px;
  }

  .deploy-info-card-text {
    width: 70px;
    text-align: right;
  }

  .deploy-input{
    width: 70%;
    border-radius:0px;
  }

  .deploy-page{
    margin-left: 5px ;
    margin-right: 5px;
    height: 100%;
  }
  .deploy-btn-submit {
    border-radius: 0;
    width: 100%;
    color: white;
    border-color: #36a3bc;
    background-color: #36a3bc;
  }
  .deploy-btn-submit:hover,
  .deploy-btn-submit:active {
    background-color: #36a3bc;
    color: white;
  }
  .deploy-card{
    margin-top: 5px;
  }
  .card{
    max-width: 100% !important;
    height: 100% !important;
    border-radius:0;
  }
  .card-header{
    background-color: white;
    text-align: center;
    color: #36a3bc;
    padding-top: 6px !important;
    padding-bottom: 6px !important;
  }
  .card-body {
    padding-top: 15px !important;
    padding-bottom: 0 !important;
  }
  .card-last-body {
    padding-bottom: 15px !important;
  }
  .card-text{
    margin-bottom: 2px;
  }
  .card-text-center{
    text-align: center;
  }

  .card-info{
    height: 35%;
    margin-top: -65px;
    padding-top: 69px;
  }
  .card-fee{
    height: 45%;
  }
  .card-result{
    height: 20%;
    margin-bottom: -26px;
    padding-bottom: 26px;
  }

  .deploy-info-card-text {
    width: 70px;
    text-align: right;
  }

  .deploy-input{
    width: 70%;
    border-radius:0px;
  }

  .deploy-custom-file-input {
    width: 100%;
  }
  .deploy-custom-file-label {
    position: absolute;
    top: 0;
    right: 0;
    left: 0;
    height: 36px;
    padding: .5rem 1.1rem;
    line-height: 1.5;
    color: #495057;
    background-color: #fff;
    border: 1px solid #ced4da;
    border-left: 1px solid #BCBEC1;
    margin-bottom: 0px;
  }
  .deploy-custom-file-label::after {
    position: absolute;
    top: 0;
    right: 0;
    bottom: 0;
    display: block;
    padding: .5rem .75rem;
    color: #495057;
    content: "Browse";
    background-color: #e9ecef;
    border-left: 1px solid #BCBEC1;
    min-width: 55px;
    text-align: center;
  }
  .deploy-custom-zh-text-file-label {
    position: absolute;
    top: 0;
    right: 0;
    left: 0;
    height: 36px;
    padding: .5rem 1.1rem;
    line-height: 1.5;
    color: #495057;
    background-color: #fff;
    border: 1px solid #ced4da;
    border-left: 1px solid #BCBEC1;
    margin-bottom: 0px;
  }
  .deploy-custom-zh-text-file-label::after {
    position: absolute;
    top: 0;
    right: 0;
    bottom: 0;
    display: block;
    padding: .5rem .75rem;
    color: #495057;
    content: "浏览";
    background-color: #e9ecef;
    border-left: 1px solid #BCBEC1;
    min-width: 55px;
    text-align: center;
  }
  .deploy-dialog-btn{
    background-color: #36a3bc;
    border-radius:0px;
  }
  .deploy-err-message {
    color: #ff0264 !important;
  }
  .input-group-text{
    border-radius:0px;
  }
  .devlop-custom-file{
    height: 36px;
  }
  .deploy-input-group-append{
    min-width: 55px;
  }
  .deploy-btn-submit-wallet{
    width: 49%;
    display: inline-block;
  }
</style>
