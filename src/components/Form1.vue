<template>
  <Form @submit.preventDefault="onSubmit">
    <div class="card">
      <div class="card-content left-align">
        <span class="card-title">
          <i class="material-icons prefix">collections</i>
          量測紀錄
        </span>
        <br />
        <div class="row">
          <div class="input-field col s12">
            <i class="material-icons prefix">account_circle</i>
            <Field
              name="userName"
              as="input"
              type="text"
              rules="required"
              v-model="registerForm.userName"
            />
            <label for="userName">姓名</label>
            <ErrorMessage name="userName" class="error" />
          </div>
        </div>
        <div class="row">
          <div class="input-field col s12">
            <i class="material-icons prefix">date_range</i>
            <Field
              name="birthDate"
              as="input"
              type="text"
              v-model="registerForm.birthDate"
              class="datepicker"
              rules="required"
            />
            <label for="birthDate">生日</label>
            <ErrorMessage name="birthDate" class="error" />
          </div>
        </div>
        <div class="row">
          <div class="input-field col s12">
            <i class="material-icons prefix">date_range</i>
            <Field
              name="inputDate"
              as="input"
              type="text"
              v-model="registerForm.inputDate"
              class="datepicker"
              rules="required"
            />
            <label for="inputDate">使用日期</label>
            <ErrorMessage name="inputDate" class="error" />
          </div>
        </div>
        <div class="row">
          <ul class="collapsible popout">
            <li>
              <div class="collapsible-header">
                <i class="material-icons">trending_up</i>使用前
              </div>
              <div class="collapsible-body">
                <div class="input-field">
                  <i class="material-icons prefix">pool</i>
                  <Field
                    name="moisture1"
                    as="input"
                    type="text"
                    v-model="registerForm.moisture1"
                    rules="required"
                  />
                  <label for="moisture1">水分</label>
                  <ErrorMessage name="moisture1" class="error" />
                </div>
                <div class="input-field">
                  <i class="material-icons prefix">tonality</i>
                  <Field name="oil1" as="input" type="text" v-model="registerForm.oil1" />
                  <label for="oil1">油分</label>
                  <ErrorMessage name="oil1" class="error" />
                </div>
                <div class="input-field">
                  <i class="material-icons prefix">toll</i>
                  <Field
                    name="elasticity1"
                    as="input"
                    type="text"
                    v-model="registerForm.elasticity1"
                    rules="required"
                  />
                  <label for="elasticity1">彈性</label>
                  <ErrorMessage name="elasticity1" class="error" />
                </div>
              </div>
            </li>
            <li>
              <div class="collapsible-header">
                <i class="material-icons">trending_down</i>使用後
              </div>
              <div class="collapsible-body">
                <div class="input-field">
                  <i class="material-icons prefix">pool</i>
                  <Field
                    name="moisture2"
                    as="input"
                    type="text"
                    v-model="registerForm.moisture2"
                    rules="required"
                  />
                  <label for="moisture2">水分</label>
                  <ErrorMessage name="moisture2" class="error" />
                </div>
                <div class="input-field">
                  <i class="material-icons prefix">tonality</i>
                  <Field name="oil2" as="input" type="text" v-model="registerForm.oil2" />
                  <label for="oil2">油分</label>
                  <ErrorMessage name="oil2" class="error" />
                </div>
                <div class="input-field">
                  <i class="material-icons prefix">toll</i>
                  <Field
                    name="elasticity2"
                    as="input"
                    type="text"
                    v-model="registerForm.elasticity2"
                    rules="required"
                  />
                  <label for="elasticity2">彈性</label>
                  <ErrorMessage name="elasticity2" class="error" />
                </div>
              </div>
            </li>
          </ul>
        </div>
        <div class="card-action center-align">
          <button class="btn waves-effect waves-light" type="submit">送出</button>
        </div>
      </div>
    </div>
  </Form>

  <div class="blockUI" style="display: none">
    <div class="loading">
      <img src="https://i.gifer.com/YCZH.gif" />
    </div>
  </div>
</template>
<script>
import { reactive, onMounted } from "vue";
import { Field, Form, ErrorMessage, defineRule } from "vee-validate";
import axios from "axios";
import Swal from "sweetalert2";

defineRule("required", (value) => {
  if (!value) {
    return "此欄位必填";
  }
  return true;
});

export default {
  components: {
    Field,
    Form,
    ErrorMessage,
  },
  setup(props, context) {
    let registerForm = reactive({});

    const airTableName = "BMI";

    const onSubmit = () => {
      console.log(JSON.stringify(registerForm));
      
      let data = {
        fields: {
          姓名: registerForm.userName,
          生日: registerForm.birthDate,
          使用日期: registerForm.inputDate,
          使用前水份: registerForm.moisture1,
          使用前油份: registerForm.oil1,
          使用前彈性: registerForm.elasticity1,
          使用後水份: registerForm.moisture2,
          使用後油份: registerForm.oil2,
          使用後彈性: registerForm.elasticity2,
        },
      };
      //console.log(JSON.stringify(data));

      document.querySelector('.blockUI').setAttribute('style', 'display:inline');
      axios
        .post(`https://api.airtable.com/v0/${import.meta.env.VITE_APP_ID}/${airTableName}`, data, {
          headers: {
            Authorization: 'Bearer ' + import.meta.env.VITE_APP_KEY,
            'Content-Type': 'application/json',
          },
        })
        .then((response) => {
          document.querySelector('.blockUI').setAttribute('style', 'display:none');
          if (!!response && !!response.data.id) {
            Swal.fire({
              icon: 'success',
              title: '儲存成功',
              html: '感謝您的填寫!',
              willClose: () => {
                window.location.reload();
              },
            });
          }
        })
        .catch((error) => {
          document.querySelector('.blockUI').setAttribute('style', 'display:none');
          Swal.fire({
            icon: 'error',
            title: '儲存失敗',
            text: '請洽管理員！',
          });
        });
        
    };

    const initComponent = async () => {
      initDatePicker();
      initCollapsible();
    };

    const initDatePicker = () => {
      const elems = document.querySelectorAll(".datepicker");
      M.Datepicker.init(elems, {
        defaultDate: new Date(),
        format: "yyyy/mm/dd",
        container: "body",
        yearRange: [1928, new Date().getFullYear() + 2],
        i18n: {
          months: [
            "一月",
            "二月",
            "三月",
            "四月",
            "五月",
            "六月",
            "七月",
            "八月",
            "九月",
            "十月",
            "十一月",
            "十二月",
          ],
          monthsShort: [
            "1月",
            "2月",
            "3月",
            "4月",
            "5月",
            "6月",
            "7月",
            "8月",
            "9月",
            "10月",
            "11月",
            "12月",
          ],
          weekdays: [
            "星期一",
            "星期二",
            "星期三",
            "星期四",
            "星期五",
            "星期六",
            "星期日",
          ],
          weekdaysShort: ["周日", "周一", "周二", "周三", "周四", "周五", "周六"],
          cancel: "取消",
          done: "確認",
        },
        autoClose: true,
      });
    };

    const initCollapsible = () => {
      const elems = document.querySelectorAll(".collapsible");
      M.Collapsible.init(elems, {
        accordion: false,
      });
    };

    onMounted(initComponent);
    return {
      onSubmit,
      registerForm,
    };
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
select {
  display: inherit;
}
#lineForm {
  padding: 5px 1rem;
}
.card {
  max-width: 512px;
  margin: 0.5em auto;
}
button {
  background-color: #4285f4;
}
.p-l-5 {
  padding: 5px;
}
.input-field {
  margin-top: 10px;
}
.radio--group p {
  padding: 5px 0;
}
.error {
  color: #f44336;
}

.blockUI {
  position: fixed;
  width: 100%;
  height: 100%;
  left: 0;
  top: 0;
  background: rgba(255, 255, 255, 0.7);
  z-index: 999;
}

.loading {
  max-width: 90%;
  max-height: 90%;
  width: auto;
  height: auto;
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
}
</style>
