<template>
  <div>
    <div style="position: relative;" class="margin">
    <canvas id="overlay"/>
    <video id="video" width="720" height="560" autoplay="true" muted ></video>
    <!-- <img :src="require('@/assets/obama.jpg')" alt="eduard" class="ward" id="input"/> -->
    </div>
    <h4>{{age}} -{{gender}}</h4>
    <v-btn large @click="face()">Detect Face</v-btn>
  </div>
</template>

<script>
import * as faceapi from 'face-api.js'
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data(){
    return{
      age:null,
      gender:null,
      expression:''
    }
  },
  mounted(){
    this.run()
    this.startVideo()

    const video = document.getElementById('video')
    const canvas = document.getElementById('overlay')
      const displaySize = {width: video.width,height:video.height}
     faceapi.matchDimensions(canvas, displaySize)
      video.addEventListener('playing',() =>{
      setInterval(async () => {
        const detections = await faceapi.detectSingleFace(video).withFaceLandmarks();
        console.log(detections)
        const resizedDetections  =faceapi.resizeResults(detections,displaySize)
        canvas.getContext('2d').clearRect(0,0,canvas.width,canvas.width)
        faceapi.draw.drawDetections(canvas, resizedDetections)
        faceapi.draw.drawFaceLandmarks(canvas, resizedDetections)
      }, 100);
    })
  },
  methods:{
    async run(){
      const facenets = await faceapi.nets
      await faceapi.nets.ssdMobilenetv1.loadFromUri('/weights')
      await faceapi.nets.faceLandmark68Net.loadFromUri('/weights')
      await faceapi.nets.faceRecognitionNet.loadFromUri('/weights')
      await faceapi.nets.faceExpressionNet.loadFromUri('/weights')
      await faceapi.nets.ageGenderNet.loadFromUri('/weights')
      await faceapi.nets.tinyFaceDetector.loadFromUri('/weights')
    
     console.log(facenets)
    },
    async startVideo(){
      navigator.mediaDevices.getUserMedia({video:{}})
      .then(function(stream){
        // eslint-disable-next-line no-undef
        video.srcObject = stream
      })
      .catch(err =>{
        console.log(err)
      })
    },
    async face(){
      const video = document.getElementById('video')
      video.addEventListener('playing',() =>{
      setInterval(async () => {
        const detections = await faceapi.detectAllFaces(video, new faceapi.TinyFaceDetectorOptions()).withFaceLandmarks().withFaceDescriptors()

        if(detections.length){
          console.log('Nc one')
        }
        console.log(detections)
      }, 1000);
    })
    },
    async detectFace(){
      //const image = document.getElementById('input')
      const image = document.getElementById('video')
      const result = await faceapi.detectAllFaces(image).withFaceLandmarks()
      const displaySize = { width: image.width, height: image.height }
      const canvas = document.getElementById('overlay')
      await faceapi.matchDimensions(canvas, displaySize)

      // resize the detected boxes in case your displayed image has a different size than the original
      const resizedDetections = await faceapi.resizeResults(result, displaySize)
      // draw detections into the canvas
      await faceapi.draw.drawDetections(canvas, resizedDetections)
      //const detection = await faceapi.detectSingleFace(image).withAgeAndGender().withFaceExpressions().withFaceLandmarks().withFaceDescriptor()
      console.log('result',result)

      // this.age = detection.age
      // this.expression = detection.expressions
      // this.gender = detection.gender
    }

  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.ward{
  height: 100%;
  width: 500px;
}

canvas{
  position:absolute;
}
</style>
