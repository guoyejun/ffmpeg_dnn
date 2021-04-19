From openvino/ubuntu18_data_dev:2021.3

USER root

RUN apt-get update
RUN apt-get install -y git gcc nasm wget vainfo clinfo vim mediainfo yasm pkg-config make libfontconfig1-dev libx264-dev

WORKDIR /workspace

RUN mkdir tensorflow && cd tensorflow && \
    wget https://storage.googleapis.com/tensorflow/libtensorflow/libtensorflow-cpu-linux-x86_64-2.4.0.tar.gz && \
    tar zxvf libtensorflow-cpu-linux-x86_64-2.4.0.tar.gz

ENV LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/intel/openvino/inference_engine/lib/intel64
ENV LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/intel/openvino/inference_engine/external/tbb/lib
ENV LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/intel/openvino/deployment_tools/ngraph/lib
ENV LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/workspace/tensorflow/lib

COPY videos/cici.mp4 cici.mp4
COPY models/openvino/2021.1/* ./
COPY models/tensorflow/v2/ssd_mobilenet_v2_coco.pb .

RUN git clone --depth=1 https://github.com/FFmpeg/FFmpeg ffmpeg

RUN cd ffmpeg && \
    mkdir build && cd build && \
    ../configure \
        --enable-gpl --enable-libx264 \
        --enable-libtensorflow \
        --enable-libopenvino \
        --extra-cflags="-I/workspace/tensorflow/include -I/opt/intel/openvino/inference_engine/include/" \
        --extra-ldflags="-L/workspace/tensorflow/lib -L/opt/intel/openvino/inference_engine/lib/intel64" \
        --enable-libfontconfig --enable-libfreetype && \
    make -j$(nproc) && \
    make install

#RUN rm -rf /workspace/ffmpeg

RUN echo 'ffmpeg -i cici.mp4 -vf dnn_detect=dnn_backend=openvino:model=face-detection-adas-0001.xml:input=data:output=detection_out:confidence=0.6:labels=face-detection-adas-0001.label,dnn_classify=dnn_backend=openvino:model=emotions-recognition-retail-0003.xml:input=data:output=prob_emotion:confidence=0.3:labels=emotions-recognition-retail-0003.label:target=face,drawbox=box_source=side_data_detection_bboxes:t=5:color=red,drawtext=text_source=side_data_detection_bboxes:fontcolor=red:fontsize=30 -an -c:v libx264 -y faces.mp4' > detect_face_classify_emotion_with_openvino.sh
RUN echo 'echo Please check the transcoding result faces.mp4' >> detect_face_classify_emotion_with_openvino.sh
RUN echo 'echo You may run \"docker cp {container_id}:/workspace/faces.mp4 /tmp/\" to download it to host machine' >> detect_face_classify_emotion_with_openvino.sh
RUN chmod +x detect_face_classify_emotion_with_openvino.sh

RUN echo 'ffmpeg -i cici.mp4 -vf dnn_detect=dnn_backend=tensorflow:input=image_tensor:output="num_detections&detection_scores&detection_classes&detection_boxes":model=ssd_mobilenet_v2_coco.pb,drawbox=box_source=side_data_detection_bboxes:t=5:color=red,drawtext=text_source=side_data_detection_bboxes:fontcolor=red:fontsize=30 -c:v libx264 -y objects.mp4' > detect_objects_with_tensorflow.sh
RUN echo 'echo Please check the transcoding result objects.mp4' >> detect_objects_with_tensorflow.sh
RUN echo 'echo You may run \"docker cp {container_id}:/workspace/objects.mp4 /tmp/\" to download it to host machine' >> detect_objects_with_tensorflow.sh
RUN chmod +x detect_objects_with_tensorflow.sh
