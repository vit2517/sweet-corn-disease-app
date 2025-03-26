<template>
  <q-page class="q-pa-md">
    <q-card>
      <q-card-section>
        <div class="q-gutter-md">
          <div class="row items-center q-mb-md">
            <!-- Video and Canvas Elements -->
            <video ref="video" autoplay playsinline class="q-mb-md" style="width: 100%; max-height: 300px;"></video>
            <canvas ref="canvas" style="display: none;"></canvas>
            <q-btn @click="captureImage" label="Capture Image" color="primary" class="full-width" />
          </div>
          <!-- Capture -->
          <div class="row items-center q-mt-md">
            <q-btn v-if="imageSrc" @click="uploadImage" label="Upload Image" color="secondary" class="full-width" />
            <q-img v-if="imageSrc" :src="imageSrc" class="q-mb-md" style="width: 100%; max-height: 300px;" />
            {{ result_images }}
          </div>
          <!-- File picker and image previews -->
          <div class="row items-center q-mt-md">
            <q-file v-model="image" label="Pick one file" filled class="full-width"
              @update:model-value="handleUpload" />
            <q-img v-if="imageUrl" :src="imageUrl" spinner-color="white" class="q-mb-md"
              style="width: 100%; max-height: 300px;" />
            {{ result_files }}
          </div>
          <div class="q-pa-md">
            <q-list>
              <q-item>
                <q-item-section>
                  <q-item-label>สาเหตุ:</q-item-label>
                  <q-item-label caption lines="2">{{ Caused_by }}</q-item-label>
                </q-item-section>
              </q-item>

              <q-separator spaced inset />

              <q-item>
                <q-item-section>
                  <q-item-label>อาการ:</q-item-label>
                  <q-item-label caption> {{ Symptoms }}</q-item-label>
                </q-item-section>
              </q-item>

              <q-separator spaced inset />

              <q-item>
                <q-item-section>
                  <q-item-label>ผลกระทบ:</q-item-label>
                  <q-item-label caption> {{ Impact }}</q-item-label>
                </q-item-section>
              </q-item>

              <q-separator spaced inset />

              <q-item>
                <q-item-section>
                  <q-item-label>การควบคุม:</q-item-label>
                  <q-item-label caption> {{ Management }}</q-item-label>
                </q-item-section>
              </q-item>
            </q-list>
          </div>
        </div>
      </q-card-section>
    </q-card>
  </q-page>
</template>

<script>
import axios from 'axios';
import { ref } from 'vue';

export default {
  data() {
    return {
      imageSrc: null,
    };
  },
  setup() {
    const image = ref(null);
    const imageUrl = ref('');
    const ModelUrl = ref('https://corn-disease-38gp.onrender.com/predict_image');
    const result_files = ref('');
    const result_images = ref('');
    const Caused_by = ref('');
    const Symptoms = ref('');
    const Impact = ref('');
    const Management = ref('');

    // Function to check the disease prediction
    const checkDiseasePrediction = (predictedLabel) => {
      // Check each disease with an if statement

      if (predictedLabel == 'CDM') {
        console.log('Prediction: Corn Disease - CDM');
        Caused_by.value = "โรคราน้าค้าง เกิดจากเชื้อรา Peronosclerospora sorghi พบแพร่ระบาดช่วงต้นฤดูฝนถึงสิ้นฤดูฝน เชื้อจะสร้างสปอร์เป็นผงสีขาวบนผิวใบช่วงเช้ามืดที่มีความชื้นสูงอากาศข้างเย็น สปอร์แก่จะแพร่กระจายโดยลมเข้าทำลายข้าวโพดต้นอื่น";
        Symptoms.value = "ลักษณะอาการ พบจุดสีขาวหรือสีเหลืองอ่อนบนใบเลี้ยงและใบจริง จุดจะขยายออกเป็นสีขาวลุกลามไปยังโคนใบ พบลายทางสีขาวเขียวอ่อนหรือเหลืองอ่อนที่โคนใบถึงปลายใบข้าวโพดจะเสียหายมาก";
        Impact.value = "ผลกระทบ ต้นกล้าจะแห้งตายและต้นโตจะแห้งตายก่อนออกดอกหรือฝัก พันธุ์ที่อ่อนแอออกดอกแต่จะไม่มีฝักหรือฝักไม่สมบูรณ์มีเมล็ดน้อยหรือไม่มีเมล็ดดอกแตกออกเป็นพุ่ม";
        Management.value = "การป้องกัน หลีกเลี่ยงการปลูกช่วงฝนตกชุก ใช้พันธุ์ต้านทาน ใช้สารเคมี Metalaxyl(Apron35DS)ในอัตรา 7-10 กรัมต่อเมล็ด 1 กิโลกรัม หรือ Metalaxyl-M 35 เปอร์เซ็นต์ ES อัตรา 3.5 มิลลิลิตรต่อเมล็ด 1 กิโลกรัม คลุกเมล็ดก่อนปลูกหรือใช้พันธุ์ต้านทาน";
        return 'CDM (Corn Damping-Off Disease)';
      } else if (predictedLabel == 'HT') {
        console.log('Prediction: Corn Disease - HT');
        Caused_by.value = "ไม่พบการระบาดของโรค ใบข้าวโพดมีสีเขียงสด";
        Symptoms.value = "ข้าวโพดมีสีเขียงสดไม่มีโรคและแมลง";
        Impact.value = "ใบข้าวโพดสีเขียวสดไม่แสดงอาการ";
        Management.value = "ป้องกันไม่ให้เกิดโรค";
        return 'HT (Hail Damage)';
      } else if (predictedLabel == 'MDMV') {
        console.log('Prediction: Corn Disease - MDMV');
        Caused_by.value = "โรคใบด่างแคระ เกิดจากเซื้อไวรัส Maize dwarf mosaic virus แพร่ระบาดโดยอาศัยเพลี้ยอ่อนข้าวโพด เพลี้ยอ่อนหญ้าดูดเชื้อจากต้นเป็นโรคไปถ่ายทอดสู่อีกต้นปกติ";
        Symptoms.value = "ลักษณะอาการ ใบเหลืองซีดทั่วทั้งใบ ยอดอ่อนมีสีเหลืองซีดหรือมีจุดปะเหลือง ต้นแคระแกร็น ต้นกล้าจะแห้งตาย ต้นโตให้ฝักไม่สมบูรณ์";
        Impact.value = "ผลกระทบ ต้นข้าวโพดติดเชื้อในช่วงการเจริญเติบโตทำให้ผลผลิตลดลง พันธุ์ที่อ่อนแอไม่ให้ผลผลิต";
        Management.value = "การป้องกัน ใข้สารเคมีเอ็กซ์มาลา มาลาไทออน(malathion)83% W/V EC ป้องกันกำจัดเพลี้ยอ่อนข้าวโพดและเพลี้ยอ่อนที่เป็นพาหะของโรคใบด่างแคระ ใช้พันธุ์ข้าวโพดที่ต้านทานต่อโรค";
        return 'MDMV (Maize Dwarf Mosaic Virus)';
      } else if (predictedLabel == 'NCLB') {
        console.log('Prediction: Corn Disease - NCLB');
        Caused_by.value = "โรคใบไหม้แผลใหญ่ เกิดจากเชื้อรา Exserohilum turcicum แพร่ระบาดตลอดฤดูกาลปลูก เชื้อสร้างสปอร์แพร่โดยลมหรือฝนเมื้อมีความซื้นสปอร์จะงอกข้าวทำลายใบข้าวโพดต้นอื่นหรือเชื้อติดไปกับเมล็ด";
        Symptoms.value = "ลักษณะอาการ ใบ กาบใบ ลาต้น และฝัก เป็นแผลขนาดใหญ่สีเทา สีน้าตาล อาการรุนแรงใบไหม้และแห้งตาย";
        Impact.value = "ผลกระทบ การติดเชื้ออย่างรุนแรงใบไหม้และแห้งตายพื้นที่การสังเคราะห์แสงของใบมีน้อยทำให้ผลผลิตลดลง ถ้าติดเชื้อก่อนออกไหมผลผลิตลดลงมากกว่าครึ่ง";
        Management.value = "การป้องกัน ใช้พันธุ์ข้าวโพดที่ต้นทาน ใช้สารเคมี ออติวา (ไดฟีโนโคนาโซล difenoconazole + อะซอกซีสโตรบินazoxystrobin) และการหมุนเวียนพืช";
        return 'NCLB (Northern Corn Leaf Blight)';
      } else if (predictedLabel == 'SCLB') {
        console.log('Prediction: Corn Disease - SCLB');
        Caused_by.value = "โรคใบไหม้แผลเล็ก เกิดจากเชื้อรา Bipolaris maydis พบระบาดมากช่วงปลายฤดูฝน เชื้อสร้างสปอร์แพร่โดยลมหรือฝนเข้าทาลายข้าวโพดต้นอื่น เซื้อติดไปกับเมล็ด";
        Symptoms.value = "ลักษณะอาการ ใบเเกิดแผลเล็กมีสีเขียวอ่อนที่ขอบใบมีสีน้ำตาลเทา ติดเซื้อรุนแรงแผลขยายรวมกันเป็นแผลใหญ่ทาให้ใบแห้งตาย";
        Impact.value = "ผลกระทบ ลดผลผลิตได้อย่างมากถ้าเกิดกรติดเซื้นในช่วงต้นฤดูกาลปลูกที่อากาศร้อนและมีความชื้น";
        Management.value = "การป้องกัน ใช้สารเคมีไตรโฟรีน triforine 20 (ซาพรอล) ใช้พันธุ์ข้าวโพดที่ทนทาน";
        return 'SCLB (Southern Corn Leaf Blight)';
      } else if (predictedLabel == 'SCMV') {
        console.log('Prediction: Corn Disease - SCMV');
        Caused_by.value = "โรคใบด่างอ้อย เกืดจากเซื้อไวรัส Sugarcane mosaic virus แพร่ระบาดด้วยเพลี้ยอ่อนเป็นพาหะนำโรค ";
        Symptoms.value = "ลักษณะอาการ ข้าวโพดมีอาการใบด่างลายเขียวซีดสลับเขียวเข้มขนานไปกับความยาวของเส้นกลางใบ ในระยะต้นโตพบเปลือกหุ้มฝักข้าวโพดเป็นสีขาวตั้งแต่ฝักเล็กจนถึงฝักโหญ่";
        Impact.value = "ผลกระทบ ผลิตลดลงประมาณครึ่งหนึ่งและทำให้ผลผลิตมีคุณภาพต่ำ";
        Management.value = "การป้องกัน ใช้สารเคมี เอ็กซ์มาลา มาลาไทออน (malathion) ป้องกันกำจัดเพลี้ยอ่อนข้าวโพด ใช้พันธุ์ข้าวโพดที่ต้านทาน";
        return 'SCMV (Sugarcane Mosaic Virus)';
      } else if (predictedLabel == 'SR') {
        console.log('Prediction: Corn Disease - SR');
        Caused_by.value = "โรคราสนิม เกืดจากเซื้อรา Puccinia polysora Undrew แพร่ระบาดช่วงปลายฤดูฝนต้นฤดูหนาว เชื้อราแพร่ระบาดด้วยอาศัยพืชที่มีชีวิตหรือส่วนของพืชที่ยังมีชีวิตตกลงบนใบต้นอื่น";
        Symptoms.value = "ลักษณะอาการ ใบ ลำต้น กาบใบ ฝัก ช่อดอกตัวผู้ แสดงอาการเป็นมีจุดนูนเล็กสีน้าตาลแดง ถ้าเป็นโรครุนแรงจะทำให้ใบแห้งตาย";
        Impact.value = "ผลกรทบ ผลผลิตลดลงประมาณครึ่งหนึ่ง น้ำหนักของฝักและจำนวนฝักลดลง";
        Management.value = "การป้องกัน ใช้สารเคมี สกอร์ (ไดพีโนคลอนาโซล difenoconazole25%) ใช้พัธุ์ต้านทาน กาจัดวัชพืชและเผาทำลายต้นพืชที่เป็นโรค";
        return "SR (Stewart's Wilt)";
      } else {
        // If the prediction doesn't match any known diseases, log an error
        console.error('Prediction does not match any known diseases.');
        return null;
      }
    };

    const handleUpload = async () => {
      console.log('handleUpload is triggered');
      const image_name = image.value.name;
      console.log('image.name: ', image_name);
      if (image.value) {
        imageUrl.value = URL.createObjectURL(image.value);
        const dataURL = imageUrl.value;
        const blob = await fetch(dataURL).then((res) => res.blob());
        console.log(blob);

        // Create FormData object and append the image Blob
        const formData = new FormData();
        formData.append('file', blob, image_name); // 'file' is the key here, matching Python's files parameter
        console.log(formData);

        // Retry logic for uploading the image
        const response = await makeRequestWithRetry(formData);
        if (response) {
          console.log(`Prediction_2: ${response}`);  
          result_files.value = checkDiseasePrediction(response);
          console.log('Response from server:', response.data);
        } else {
          console.error('Failed to upload image after multiple attempts');
        }
      }
    };

    // Retry mechanism for axios request
    const makeRequestWithRetry = async (formData, retries = 3, delay = 3000) => {
      const class_names = ['CDM', 'HT', 'MDMV', 'NCLB', 'SCLB', 'SCMV', 'SR'];
      for (let attempt = 0; attempt < retries; attempt++) {
        try {
          const response = await axios.post(ModelUrl.value, formData, {
            // No need for 'Content-Type': 'multipart/form-data'
            // Axios will automatically handle it when using FormData
            timeout: 10000, // Increase timeout to 10 seconds for slow responses
          });
          // Check if the response contains valid prediction data
          if (response && response.data && response.data.prediction) {
            const predictedLabel = response.data.prediction;
            // Check if the predicted label is one of the 7 diseases in the class_names array
            if (class_names.includes(predictedLabel)) {
              // If it matches a disease in the list, log and return the prediction
              console.log(`Prediction_1: ${predictedLabel}`);
              return predictedLabel;
            } else {
              // If the prediction doesn't match any disease, log an error
              console.error('Prediction does not match any known diseases.');
              return null;
            }
          } else {
            // If the response does not contain the expected data, log an error
            console.error('Invalid response data:', response.data);
            return null;
          }
        } catch (error) {
          console.error(`Attempt ${attempt + 1} failed: ${error.message}. Retrying in ${delay / 1000} seconds...`);
          await new Promise((resolve) => setTimeout(resolve, delay));
        }
      }
      console.error('Request failed after all retry attempts');
      return null;
    };
    return {
      image,
      imageUrl,
      ModelUrl,
      handleUpload,
      result_files,
      result_images,
      Caused_by,
      Symptoms,
      Impact,
      Management,
    };
  },
  mounted() {
    this.startCamera();
  },
  methods: {
    async startCamera() {
      const video = this.$refs.video;
      try {
        const stream = await navigator.mediaDevices.getUserMedia({ video: true });
        video.srcObject = stream;
        video.play();
      } catch (err) {
        console.error('Error accessing webcam:', err);
      }
    },
    captureImage() {
      const video = this.$refs.video;
      const canvas = this.$refs.canvas;

      canvas.width = video.videoWidth;
      canvas.height = video.videoHeight;

      const context = canvas.getContext('2d');
      context.drawImage(video, 0, 0, canvas.width, canvas.height);

      this.imageSrc = canvas.toDataURL('image/png');
    },
    async uploadImage() {
      // Convert the data URL to a Blob
      const dataURL = this.imageSrc;
      console.log(dataURL);
      const blob = await fetch(dataURL).then((res) => res.blob());
      console.log(blob);

      // Create FormData object and append the image Blob
      const formData = new FormData();
      formData.append('file', blob, 'captured-image.png'); // 'file' as the key, matching the Python code
      console.log(formData);

      // Retry logic for uploading the image
      const response = await this.makeRequestWithRetry(formData);
      if (response) {
        this.result_images = response.data;
        console.log('Response from server:', this.result_images);
      } else {
        console.error('Failed to upload image after multiple attempts');
      }
    },
  },
};
</script>


<style scoped>
/* Additional styles to enhance UI */
.q-page {
  display: flex;
  justify-content: center;
  align-items: center;
}

.q-card {
  width: 100%;
  max-width: 600px;
}

.q-card-section {
  padding: 20px;
}

.q-file {
  margin-top: 10px;
}

.q-img {
  border: 1px solid #ccc;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}
</style>
