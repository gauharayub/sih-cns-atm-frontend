<template>
  <Loader v-if="loading" />

  <div v-else class="body">
    <b-container fluid class="bv-example-row mt-60">
      <b-row>
        <b-col cols="12" md="8" offset-md="2" id="main-box">
          <h2>Compliance form</h2>
          <b-form @submit="onSubmit" @reset="onReset" v-if="show">
            <div class="form-group-1">
              <p>
                Assignment Date :
                <span>{{ form.assignmentDate.slice(0, 10) }}</span>
              </p>
               <p>
                Deadline Date :
                <span>{{ form.DeadlineDate.slice(0, 10) }}</span>
              </p>

              <p>
                Equipment Name : <span>{{ form.equipmentName }}</span>
              </p>

              <p>
                Description :
                <span>{{ form.description }}</span>
              </p>

              <p>
                Code : <span>{{ form.equipmentCode }}</span>
              </p>

              <p>
                Status : <span>{{ form.status || 'Progress' }}</span>
              </p>

              <p>
                Location : <span>{{ form.location }}</span>
              </p>
              <p>
                Cycle: <span>{{ form.cycle }}</span>
              </p>
            </div>

            <div class="form-group-2">
              <!-- for tasklist generation  -->
              <b-form-group>
                <h3>Task List:</h3>

                <div v-for="(task, index) in form.tasklist" :key="index">
                  <br />
                  <p># {{ task }}</p>
                  <b-form-radio-group
                    :id="`${index}`"
                    v-model="TaskListValue[index]"
                    :options="optionsTaskList"
                    buttons
                    button-variant="outline-primary"
                    size="md"
                    name="radio-btn-outline"
                  ></b-form-radio-group>
                </div>
              </b-form-group>

              <div
                id="fileSelector"
                v-for="(file, index) in files"
                :key="index"
              >
                <div class="card">
                  <div class="card-body">
                    <label for="file" size="sm">File :</label>
                    <b-form-file
                      name="file"
                      class="makeInLine"
                      v-model="fileName[index]"
                      @change="onFileChange(index, $event)"
                      :state="Boolean(file)"
                      placeholder="Choose a file or drop it here..."
                      drop-placeholder="Drop file here..."
                      size="sm"
                      accept="image/*"
                    ></b-form-file>
                    <span
                      class="pull-right clickable close-icon"
                      data-effect="fadeOut"
                      @click="deleteFileItem(index)"
                    >
                      <i class="fa fa-times"></i>
                    </span>
                    <div class="mt-3">
                      Selected file:
                      <p>{{ fileName[index] ? fileName[index].name : '' }}</p>
                    </div>
                    <div>
                      <img :src="image[index]" alt />
                    </div>
                    <b-form-group
                      id="input-group-8"
                      label="Comments:"
                      label-for="input-8"
                    >
                      <textarea
                        cols="46"
                        rows="2"
                        id="input-8"
                        v-model="capturedComment[index]"
                      ></textarea>
                    </b-form-group>
                  </div>
                </div>
                <br />
              </div>

              <div class="pull-right">
                <b-button class="mr-1" type="submit" id="submit"
                  >Submit</b-button
                >
                <b-button class="btn-danger" type="reset">Reset</b-button>
              </div>
            </div>

            <!-- for files -->
            <b-button class="mr-1" @click="addFile">Add Files</b-button>
          </b-form>
          <br />
        </b-col>
      </b-row>
    </b-container>
  </div>
</template>
<script>
import Axios from '@/methods/axiosInstance.js'
import Loader from '@/pages/Layout/Loader'

export default {
  data() {
    return {
      loading: true,
      form: {},
      optionsTaskList: [
        { text: 'Completed', value: 'done' },
        { text: 'Not Completed', value: 'not done' }
      ],
      TaskListValue: [],
      show: true,
      capturedFile: [], //{file:file, text:comments}
      capturedComment: [],
      files: 1,
      fileName: [],
      image: []
    }
  },
  components: {
    Loader
  },
  async mounted() {
    try {
      const { data } = await Axios().get(`/compliance/${this.$route.params.id}`)
      console.log(data)
      this.form = data
      this.loading = false
    } catch (error) {
      this.loading = false

      if (error.response && error.response.status === 401) {
        this.$router.push({ name: 'login' })
      }
      console.log(error)
    }
  },
  methods: {
    deleteFileItem(index) {
      this.capturedFile.splice(index)
      this.capturedComment.splice(index)
      this.fileName.splice(index)
      this.files--
    },
    onFileChange(index, e) {
      console.log(this)
      const fileReader = new FileReader()
      fileReader.onload = event => {
        console.log(event.target.result)
        image[index] = event.target.result
      }
      fileReader.readAsDataURL(e.target.files[0])
      this.capturedFile[index] = e.target.files[0]
    },
    addFile(event) {
      event.preventDefault()
      this.files++
    },
    async onSubmit(evt) {
      try {
        this.loading = true

        evt.preventDefault()
        const fd = new FormData()
        this.capturedFile.forEach(file => fd.append('workImage', file))
        fd.append('comments', JSON.stringify(this.capturedComment))
        fd.append('taskListValue', JSON.stringify(this.TaskListValue))

        const res = await Axios().post(
          `/submit-compliance/${this.$route.params.id}`,
          fd
        )

        this.$router.push({ name: 'dashboard' })

        this.loading = false
      } catch (error) {
        this.loading = false
        console.log(error)
      }
    },
    onReset(evt) {
      evt.preventDefault()
      // Reset our form values
      this.TaskListValue = []
      this.files = 1
      this.capturedFile = []
      this.capturedComment = []
      this.fileName = []
      // Trick to reset/clear native browser form validation state
      this.show = false
      this.$nextTick(() => {
        //things changed and noticed by vue but showing changing status before rendering to dom
        this.show = true
      })
    }
  }
}
</script>
<style lang="css" scoped>
@import url('https://fonts.googleapis.com/css2?family=Roboto+Slab:wght@400;700&display=swap');
.makeInLine {
  width: 65% !important;
  margin: 0 20px;
}
.body {
  background-color: #0d1b2a;
  padding: 60px 0;
  font-family: 'Roboto Slab';
  font-size: 13px;
  line-height: 1.8;
  color: black;
  font-weight: 400;
}
.close {
  border: 1px solid grey !important;
  padding: 6px;
  position: relative;
  left: -5%;
  cursor: pointer;
}

/*close icon*/
.clickable {
  cursor: pointer;
}

textarea {
  border: 1px solid grey;
  padding: 6px 10px;
  width: 100%;
  border-radius: 10px 10px 10px 10px;
}
#submit {
  background-color: #2493e2;
  border: none;
  width: 100px;
}

#main-box {
  background-color: white;
  padding: 50px 60px 70px 60px;
  border-radius: 10px;
  opacity: 0.97;
}
h2 {
  line-height: 1.8;
  margin: 0;
  padding: 0;
  font-weight: bold;
  color: #222;
  font-family: 'Roboto Slab', serif;
  font-size: 20px;
  margin-bottom: 30px;
  text-transform: uppercase;
}
h3 {
  font-weight: bold;
  color: #222;
  font-size: 17px;
  margin: 0px;
  margin-top: 20px;
  margin-bottom: 10px;
}
input {
  width: 100%;
  display: block;
  border: none;
  border-bottom: 2px solid #ebebeb;
  padding: 5px 0;
  color: #222;
  margin-bottom: 31px;
  font-family: 'Roboto Slab';
}
.card {
  background-color: rgba(0, 0, 0, 0.1);
}
button {
  box-shadow: 5px 5px 10px #888888;
}
</style>
