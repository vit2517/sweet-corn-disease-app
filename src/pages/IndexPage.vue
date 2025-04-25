<template>
  <q-page class="q-pa-md">
    <q-card>
      <q-card-section>
        <div class="q-gutter-md">
          <!-- Camera Selection -->
          <div class="row items-center q-mb-md">
            <q-select
              v-model="selectedCamera"
              :options="cameraOptions"
              label="Select Camera"
              outlined
              dense
              class="col"
            />
            <q-btn 
              @click="switchCamera" 
              label="Switch Camera" 
              color="grey" 
              class="q-ml-md"
            />
          </div>
          
          <!-- Video and Canvas Elements -->
          <div class="row items-center q-mb-md">
            <video ref="video" autoplay playsinline class="q-mb-md" style="width: 100%; max-height: 300px;"></video>
            <canvas ref="canvas" style="display: none;"></canvas>
            <q-btn @click="captureImage" label="Capture Image" color="primary" class="full-width" />
          </div>
          
          <!-- Capture Preview -->
          <div class="row items-center q-mt-md">
            <q-btn v-if="imageSrc" @click="uploadImage" label="Upload Image" color="secondary" class="full-width" />
            <q-img v-if="imageSrc" :src="imageSrc" class="q-mb-md" style="width: 100%; max-height: 300px;" />
            {{ result_images }}
          </div>
          
          <!-- File Upload -->
          <div class="row items-center q-mt-md">
            <q-file v-model="image" label="Pick one file" filled class="full-width" @update:model-value="handleUpload" />
            <q-img v-if="imageUrl" :src="imageUrl" spinner-color="white" class="q-mb-md" style="width: 100%; max-height: 300px;" />
            {{ result_files }}
          </div>
          
          <!-- Disease Info -->
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
      selectedCamera: 'environment', // Default to rear camera
      cameraOptions: [
        { label: 'Rear Camera', value: 'environment' },
        { label: 'Front Camera', value: 'user' }
      ],
      currentStream: null,
      image: ref(null),
      imageUrl: ref(''),
      ModelUrl: ref('https://corn-disease-38gp.onrender.com/predict_image'),
      result_files: ref(''),
      result_images: ref(''),
      Caused_by: ref(''),
      Symptoms: ref(''),
      Impact: ref(''),
      Management: ref('')
    };
  },

  mounted() {
    this.startCamera();
  },

  beforeUnmount() {
    this.stopCamera();
  },

  methods: {
    async startCamera() {
      this.stopCamera(); // Stop any existing stream first

      const video = this.$refs.video;
      try {
        const constraints = {
          video: {
            facingMode: this.selectedCamera,
            width: { ideal: 1280 },
            height: { ideal: 720 }
          }
        };
        
        const stream = await navigator.mediaDevices.getUserMedia(constraints);
        this.currentStream = stream;
        video.srcObject = stream;
        video.play();
      } catch (err) {
        console.error('Error accessing camera:', err);
        // Fallback to any available camera
        try {
          const stream = await navigator.mediaDevices.getUserMedia({ video: true });
          this.currentStream = stream;
          video.srcObject = stream;
          video.play();
        } catch (fallbackErr) {
          console.error('Could not access any camera:', fallbackErr);
        }
      }
    },

    stopCamera() {
      if (this.currentStream) {
        this.currentStream.getTracks().forEach(track => track.stop());
        this.currentStream = null;
      }
    },

    switchCamera() {
      this.selectedCamera = this.selectedCamera === 'user' ? 'environment' : 'user';
      this.startCamera();
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
      if (!this.imageSrc) return;

      const blob = await fetch(this.imageSrc).then((res) => res.blob());
      const formData = new FormData();
      formData.append('file', blob, 'captured-image.png');

      const response = await this.makeRequestWithRetry(formData);
      if (response) {
        this.result_images = this.checkDiseasePrediction(response);
      } else {
        console.error('Failed to upload image');
      }
    },

    checkDiseasePrediction(predictedLabel) {
      const diseases = {
        'CDM': {
          Caused_by: "โรคราน้าค้าง เกิดจากเชื้อรา Peronosclerospora sorghi พบแพร่ระบาดช่วงต้นฤดูฝนถึงสิ้นฤดูฝน เชื้อจะสร้างสปอร์เป็นผงสีขาวบนผิวใบช่วงเช้ามืดที่มีความชื้นสูงอากาศข้างเย็น สปอร์แก่จะแพร่กระจายโดยลมเข้าทำลายข้าวโพดต้นอื่น",
          Symptoms: "ลักษณะอาการ พบจุดสีขาวหรือสีเหลืองอ่อนบนใบเลี้ยงและใบจริง จุดจะขยายออกเป็นสีขาวลุกลามไปยังโคนใบ พบลายทางสีขาวเขียวอ่อนหรือเหลืองอ่อนที่โคนใบถึงปลายใบข้าวโพดจะเสียหายมาก",
          Impact: "ผลกระทบ ต้นกล้าจะแห้งตายและต้นโตจะแห้งตายก่อนออกดอกหรือฝัก พันธุ์ที่อ่อนแอออกดอกแต่จะไม่มีฝักหรือฝักไม่สมบูรณ์มีเมล็ดน้อยหรือไม่มีเมล็ดดอกแตกออกเป็นพุ่ม",
          Management: "การป้องกัน หลีกเลี่ยงการปลูกช่วงฝนตกชุก ใช้พันธุ์ต้านทาน ใช้สารเคมี Metalaxyl(Apron35DS)ในอัตรา 7-10 กรัมต่อเมล็ด 1 กิโลกรัม หรือ Metalaxyl-M 35 เปอร์เซ็นต์ ES อัตรา 3.5 มิลลิลิตรต่อเมล็ด 1 กิโลกรัม คลุกเมล็ดก่อนปลูกหรือใช้พันธุ์ต้านทาน"
        },
        'HT': {
          Caused_by: "ไม่พบการระบาดของโรค ใบข้าวโพดมีสีเขียงสด",
          Symptoms: "ใข้าวโพดมีสีเขียงสดไม่มีโรคและแมลง",
          Impact: "ใบข้าวโพดสีเขียวสดไม่แสดงอาการ",
          Management: "หมั่นตรวจแปลงเพื่อป้องกันไม่ให้เกิดโรค"
        },
        'MDMV': {
          Caused_by: "โรคใบด่างแคระ เกิดจากเซื้อไวรัส Maize dwarf mosaic virus แพร่ระบาดโดยอาศัยเพลี้ยอ่อนข้าวโพด เพลี้ยอ่อนหญ้าดูดเชื้อจากต้นเป็นโรคไปถ่ายทอดสู่อีกต้นปกติ",
          Symptoms: "ลักษณะอาการ ใบเหลืองซีดทั่วทั้งใบ ยอดอ่อนมีสีเหลืองซีดหรือมีจุดปะเหลือง ต้นแคระแกร็น ต้นกล้าจะแห้งตาย ต้นโตให้ฝักไม่สมบูรณ์",
          Impact: "ผลกระทบ ต้นข้าวโพดติดเชื้อในช่วงการเจริญเติบโตทำให้ผลผลิตลดลง พันธุ์ที่อ่อนแอไม่ให้ผลผลิต",
          Management: "การป้องกัน ใข้สารเคมีเอ็กซ์มาลา มาลาไทออน(malathion)83% W/V EC ป้องกันกำจัดเพลี้ยอ่อนข้าวโพดและเพลี้ยอ่อนที่เป็นพาหะของโรคใบด่างแคระ ใช้พันธุ์ข้าวโพดที่ต้านทานต่อโรค"
        },
        'NCLB': {
          Caused_by: "โรคใบไหม้แผลใหญ่ เกิดจากเชื้อรา Exserohilum turcicum แพร่ระบาดตลอดฤดูกาลปลูก เชื้อสร้างสปอร์แพร่โดยลมหรือฝนเมื้อมีความซื้นสปอร์จะงอกข้าวทำลายใบข้าวโพดต้นอื่นหรือเชื้อติดไปกับเมล็ด",
          Symptoms: "ลักษณะอาการ ใบ กาบใบ ลาต้น และฝัก เป็นแผลขนาดใหญ่สีเทา สีน้าตาล อาการรุนแรงใบไหม้และแห้งตาย",
          Impact: "ผลกระทบ การติดเชื้ออย่างรุนแรงใบไหม้และแห้งตายพื้นที่การสังเคราะห์แสงของใบมีน้อยทำให้ผลผลิตลดลง ถ้าติดเชื้อก่อนออกไหมผลผลิตลดลงมากกว่าครึ่ง",
          Management: "การป้องกัน ใช้พันธุ์ข้าวโพดที่ต้นทาน ใช้สารเคมี ออติวา (ไดฟีโนโคนาโซล difenoconazole + อะซอกซีสโตรบินazoxystrobin) และการปลูกพืชหมุนเวียน"
        },
        'SCLB': {
          Caused_by: "โรคใบไหม้แผลเล็ก เกิดจากเชื้อรา Bipolaris maydis พบระบาดมากช่วงปลายฤดูฝน เชื้อสร้างสปอร์แพร่โดยลมหรือฝนเข้าทาลายข้าวโพดต้นอื่น เซื้อติดไปกับเมล็ด",
          Symptoms: "ลักษณะอาการ ใบเเกิดแผลเล็กมีสีเขียวอ่อนที่ขอบใบมีสีน้ำตาลเทา ติดเซื้อรุนแรงแผลขยายรวมกันเป็นแผลใหญ่ทาให้ใบแห้งตาย",
          Impact: "ผลกระทบ ผลผลิตลดลงได้อย่างมากถ้าเกิดการติดเซื้อในช่วงต้นฤดูกาลปลูกที่อากาศร้อนและมีความชื้น",
          Management: "การป้องกัน ใช้สารเคมีไตรโฟรีน triforine 20 (ซาพรอล) ใช้พันธุ์ข้าวโพดที่ต้านทาน"
        },
        'SCMV': {
          Caused_by: "โรคใบด่างอ้อย เกืดจากเซื้อไวรัส Sugarcane mosaic virus แพร่ระบาดด้วยเพลี้ยอ่อนเป็นพาหะนำโรค",
          Symptoms: "ลักษณะอาการ ข้าวโพดมีอาการใบด่างลายเขียวซีดสลับเขียวเข้มขนานไปกับความยาวของเส้นกลางใบ ในระยะต้นโตพบเปลือกหุ้มฝักข้าวโพดเป็นสีขาวตั้งแต่ฝักเล็กจนถึงฝักโหญ่",
          Impact: "ผลกระทบ ผลิตลดลงประมาณครึ่งหนึ่งและทำให้ผลผลิตมีคุณภาพต่ำ",
          Management: "การป้องกัน ใช้สารเคมี เอ็กซ์มาลา มาลาไทออน (malathion) ป้องกันกำจัดเพลี้ยอ่อนข้าวโพด ใช้พันธุ์ข้าวโพดที่ต้านทาน"
        },
        'SR': {
          Caused_by: "โรคราสนิม เกืดจากเซื้อรา Puccinia polysora Undrew แพร่ระบาดช่วงปลายฤดูฝนต้นฤดูหนาว เชื้อราแพร่ระบาดด้วยอาศัยพืชที่มีชีวิตหรือส่วนของพืชที่ยังมีชีวิตตกลงบนใบต้นอื่น",
          Symptoms: "ลักษณะอาการ ใบ ลำต้น กาบใบ ฝัก ช่อดอกตัวผู้ แสดงอาการเป็นมีจุดนูนเล็กสีน้าตาลแดง ถ้าเป็นโรครุนแรงจะทำให้ใบแห้งตาย",
          Impact: "ผลกรทบ ผลผลิตลดลงประมาณครึ่งหนึ่ง น้ำหนักของฝักและจำนวนฝักลดลง",
          Management: "การป้องกัน ใช้สารเคมี สกอร์ (ไดพีโนคลอนาโซล difenoconazole25%) ใช้พัธุ์ต้านทาน กาจัดวัชพืชและเผาทำลายต้นพืชที่เป็นโรค"
        },
        // ... other disease definitions ...
      };

      if (diseases[predictedLabel]) {
        const disease = diseases[predictedLabel];
        this.Caused_by = disease.Caused_by;
        this.Symptoms = disease.Symptoms;
        this.Impact = disease.Impact;
        this.Management = disease.Management;
        return predictedLabel;
      } else {
        console.error('Unknown disease prediction');
        return null;
      }
    },

    async handleUpload() {
      if (!this.image) return;

      this.imageUrl = URL.createObjectURL(this.image);
      const blob = await fetch(this.imageUrl).then((res) => res.blob());
      const formData = new FormData();
      formData.append('file', blob, this.image.name);

      const response = await this.makeRequestWithRetry(formData);
      if (response) {
        this.result_files = this.checkDiseasePrediction(response);
      } else {
        console.error('Failed to upload file');
      }
    },

    async makeRequestWithRetry(formData, retries = 3, delay = 3000) {
      const class_names = ['CDM', 'HT', 'MDMV', 'NCLB', 'SCLB', 'SCMV', 'SR'];
      
      for (let attempt = 0; attempt < retries; attempt++) {
        try {
          const response = await axios.post(this.ModelUrl, formData, {
            timeout: 10000,
          });
          
          if (response?.data?.prediction && class_names.includes(response.data.prediction)) {
            return response.data.prediction;
          }
        } catch (error) {
          console.error(`Attempt ${attempt + 1} failed:`, error);
          if (attempt < retries - 1) {
            await new Promise(resolve => setTimeout(resolve, delay));
          }
        }
      }
      return null;
    }
  },

  watch: {
    selectedCamera() {
      this.startCamera();
    }
  }
};
</script>

<style scoped>
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