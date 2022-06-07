# gaze-estimation
目線推定

※ 「動画ファイルを入力すると写っている人の目線を推定した結果を動画上に描画する処理」をプログラムを1行もかかずに実現する

## 環境設定・インストール

Ubuntu-20.04LTSを想定している

以下1から4を実行する

```
# 1. CMakeのインストール
sudo apt install cmake

# 2. Python 開発環境のインストール (3.9の部分は自身の環境に合わせる)
sudo apt install python3.9-dev

# 3. ptgazeのインストール
pip install ptgaze 

# 4. protobuf がバージョン不一致でエラーになるのでダウングレード
pip install -U protobuf~=3.20.0
```
以上で設定は完了

### [オプション]バウンディングボックス描画のオフ設定

デフォルトでは判定された顔にバウンディングボックスが描かれるのでオフにしたい場合はこの記述にしたがって設定する。
なお、ptgazeではこの設定はコマンドラインオプションで設定できないため、直接設定ファイルを修正する。

パッケージファイルがインストールされたディレクトリ(例: {venv}/lib/python3.9/site-packages/ptgaze/data/configs/)から下記3ファイルの設定を1か所書き換える

- eth-xgaze.yaml
- mpiifacegaze.yaml
- mpiigaze.yaml

書き換える箇所は、ファイル内の demo: の設定うち`show_bbox: true`を`show_bbox: false` にする

具体例:

```
demo:
  use_camera: true
  display_on_screen: true
  wait_time: 1
  image_path: null
  video_path: null
  output_dir: null
  output_file_extension: avi
  head_pose_axis_length: 0.05
  gaze_visualization_length: 0.05
  show_bbox: true
  show_head_pose: false
  show_landmarks: false
  show_normalized_image: false
```

を

```
demo:
  use_camera: true
  display_on_screen: true
  wait_time: 1
  image_path: null
  video_path: null
  output_dir: null
  output_file_extension: avi
  head_pose_axis_length: 0.05
  gaze_visualization_length: 0.05
  show_bbox: false
  show_head_pose: false
  show_landmarks: false
  show_normalized_image: false
```
に変更

## 使い方(ptgaze)

```
usage: ptgaze [-h] [--config CONFIG] [--mode {mpiigaze,mpiifacegaze,eth-xgaze}]
              [--face-detector {dlib,face_alignment_dlib,face_alignment_sfd,mediapipe}] [--device {cpu,cuda}]
              [--image IMAGE] [--video VIDEO] [--camera CAMERA] [--output-dir OUTPUT_DIR] [--ext {avi,mp4}]
              [--no-screen] [--debug]

optional arguments:
  -h, --help            show this help message and exit
  --config CONFIG       Config file. When using a config file, all the other commandline arguments are ignored. See
                        https://github.com/hysts/pytorch_mpiigaze_demo/ptgaze/data/configs/eth-xgaze.yaml
  --mode {mpiigaze,mpiifacegaze,eth-xgaze}
                        With 'mpiigaze', MPIIGaze model will be used. With 'mpiifacegaze', MPIIFaceGaze model will be
                        used. With 'eth-xgaze', ETH-XGaze model will be used.
  --face-detector {dlib,face_alignment_dlib,face_alignment_sfd,mediapipe}
                        The method used to detect faces and find face landmarks (default: 'mediapipe')
  --device {cpu,cuda}   Device used for model inference.
  --image IMAGE         Path to an input image file.
  --video VIDEO         Path to an input video file.
  --camera CAMERA       Camera calibration file. See
                        https://github.com/hysts/pytorch_mpiigaze_demo/ptgaze/data/calib/sample_params.yaml
  --output-dir OUTPUT_DIR, -o OUTPUT_DIR
                        If specified, the overlaid video will be saved to this directory.
  --ext {avi,mp4}, -e {avi,mp4}
                        Output video file extension.
  --no-screen           If specified, the video is not displayed on screen, and saved to the output directory.
  --debug
```

## 動画を入力にして目線推定する処理の例

```
# MPIIGaze Model を使用する場合
$ ptgaze --mode mpiigaze --no-screen --video input.avi

# MPIIFaceGaze Model を使用する場合
$ ptgaze --mode mpiifacegaze --no-screen --video input.avi 

# ETH-XGaze Model を使用する場合
$ ptgaze --mode eth-xgaze --no-screen --video input.avi 
```

CUDAが設定されている場合は`--device cuda`オプションでGPU使用設定が有効になる

例:

```
$ ptgaze --device cuda --mode eth-xgaze --no-screen --video input.avi 
```

いずれも outputs の中に目線推定された動画ファイルが作成される(下記動画の水色の線が推定された視線)

https://user-images.githubusercontent.com/91955493/172399342-7e97ebac-358b-434c-91d7-d7bb94b73696.mp4

## 使用したシステムの解説

この処理は、[A PyTorch implementation of MPIIGaze and MPIIFaceGaze](https://github.com/hysts/pytorch_mpiigaze)のデモプログラムである、[A demo program of gaze estimation models (MPIIGaze, MPIIFaceGaze, ETH-XGaze)](https://github.com/hysts/pytorch_mpiigaze_demo)を使用して実現している

目線推定モデルとしては、下記の三種類が使用できる

1. MPIIGaze (--mode mpiigaze)
2. MPIIFaceGaze (--mode mpiifacegaze)
3. ETH-XGaze (--mode eth-xgaze)

顔認識モデルとしては、下記の四種類が使用できる(デフォルトはmediapipe)

1. dlib (--face-detector dlib)
2. face_alignment_dlib (--face-detector face_alignment_dlib)
3. face_alignment_sfd (--face-detector face_alignment_sfd)
4. mediapipe (--face-detector mediapipe)

以上
