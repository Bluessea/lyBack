<template>
  <v-form v-model="valid" ref="brandForm">
    <v-text-field
      label="品牌名称"
      v-model="brand.name"
      :rules="[v => !!v || '品牌名称不能为空']"
      :counter="10"
      required
    />
    <v-text-field
      label="首字母"
      v-model="brand.letter"
      :rules="[v => v.length === 1 || '首字母只能是1位']"
      required
      mask="A"
    />
    <v-cascader url="/item/category/list" required
                v-model="brand.categories"
                multiple label="商品分类"/>
    <v-layout row>
      <v-flex xs3>
        <span style="font-size: 16px; color: #444">品牌LOGO：</span>
      </v-flex>
      <v-flex>
        <v-upload
          v-model="brand.image" url="/upload/image" :multiple="false" :pic-width="250" :pic-height="90"
        />
      </v-flex>
    </v-layout>
    <v-layout class="my-4">
      <v-btn @click="submit" color="primary">提交</v-btn>
      <v-btn @click="clear" color="warning">重置</v-btn>
    </v-layout>
  </v-form>
</template>

<script>
  import config from '@/config';
  export default {
    name: "brand-form",
    props: {
      oldBrand: Object,
      isEdit: {
        type: Boolean,
        default: false
      },
      show: {
        type: Boolean,
        default: true
      }
    },
    data() {
      return {
        baseUrl: config.api,
        valid:false,
        brand: {
          name: "",
          image: "",
          letter: "",
          categories: []
        },
        nameRules:[
          v=>!!v || "品牌名称不能为空",
          v=> v.length >1 || "品牌名称至少2位"
        ],
        letterRules:[
          v=>!!v || "首字母不能为空",
          v=> /^[a-zA-Z]{1}$/.test(v) || "品牌名称至少2位"
        ],
        imageDialogVisible:false
      }
    },
    watch: {
      oldBrand:{
        deep:true,
        handler(val){
          Object.deepCopy(val,this.brand);
        }
      }
    },
    methods: {
      submit() {
        // 表单校验
        if (this.$refs.brandForm.validate()) {
          //this.brand.categories = this.brand.categories.map(c => c.id);
          //this.brand.letter = this.brand.letter.toUpperCase();
          //定义请求参数对象，通过解构表达式来获取brand中的属性
          const {categories,letter,...params} = this.brand;  //params是个对象 用于保存brand中除categories和letter外的参数
          //数据库中只保存分类的id即可，因此我们对categories的值进行处理，只保留id并保存为字符串
          params.cids = categories.map(c => c.id).join(",");
          //将字母都处理为大写
          params.letter = letter.toUpperCase();  //现在params中属性有{name,imagem,cids,letter}
          // 将数据提交到后台
          this.$http({
            method: this.isEdit ? 'put' : 'post',
            url: '/item/brand',
            data: this.$qs.stringify(params)
            //data:params
          }).then(() => {
            // 关闭窗口
            this.$message.success("保存成功！");
            this.closeWindow();
          }).catch(() => {
            this.$message.error("保存失败！");
          }); 
        }
      },
      clear() {
        // 重置表单
        this.$refs.brandForm.reset();
        this.categories = [];
      },
      // 图片上传出成功后操作
      handleImageSuccess(res) {
        this.brand.image = res;
      },
      removeImage(){
        this.brand.image = "";
      },
      closeWindow(){
        this.$emit("close");
       
      }
    }
  }
</script>

<style scoped>

</style>
