# 本文档用于阐明vilib中的用途，以及里面各类函数的使用

## 一.主体可以看作两部分，一部分为获取和处理帧的线程（Vilib.camera（）函数），第二部分为把处理完的帧通过falsk这个框架，实现局域网内的视频流传输（video_feed（）函数），通过Vilib.camera_start(web_func = True,inverted_flag = False)这个函数同时启动上述的两个线程，然后打开网址输入：IP + :9000/mjpg，例如IP是192.168.18.146，则输入192.168.18.146:9000/mjpg

## 二.其它的Vilib这个类里面的函数，可以分为两类，一类是用作外部接口返回识别到的目标参数（如类型，长宽，数量等），第二类是类内部使用的，是在Vilib.camera()这个函数的循环里，是每个识别函数的主体部分，如颜色识别，人脸识别等,如下所示

```python
img = Vilib.gesture_calibrate(img)
img = Vilib.traffic_detect(img)
img = Vilib.color_detect_func(img)
img = Vilib.human_detect_func(img)
img = Vilib.gesture_recognition(img)
img = Vilib.qrcode_detect_func(img)
```

ter_2 = tflite.Interpreter(model_path=gesture_model_path)
interpreter_2.allocate_tensors()

# Get input and output tensors.
input_details_1 = interpreter_1.get_input_details()
# print(str(input_details_1))
output_details_1 = interpreter_1.get_output_details()
# print(str(output_details_1))


# Get input and output tensors.
input_details_2 = interpreter_2.get_input_details()
# print(str(input_details_2))
output_details_2 = interpreter_2.get_output_details()
# print(str(output_details_2))


app = Flask(__name__)
@app.route('/')
def index():
    """Video streaming home page."""
    return render_template('index.html')

def get_frame():
    return cv2.imencode('.jpg', Vilib.img_array[0])[1].tobytes()


def get_qrcode_pictrue():
    return cv2.imencode('.jpg', Vilib.img_array[1])[1].tobytes()

def get_png_frame():
    return cv2.imencode('.png', Vilib.img_array[0])[1].tobytes()

def gen():
    """Video streaming generator function."""
    while True:  

        frame = get_frame()
        yield (b'--frame\r\n'
               b'Content-Type: image/jpeg\r\n\r\n' + frame + b'\r\n')
        time.sleep(0.03)

@app.route('/mjpg')   ## video
def video_feed():
    # from camera import Camera
    """Video streaming route. Put this in the src attribute of an img tag."""
    response = Response(gen(),
                    mimetype='multipart/x-mixed-replace; boundary=frame') 
    response.headers.add("Access-Control-Allow-Origin", "*")
    return response

@app.route('/mjpg.jpg')  ##picture
def video_feed_jpg():
    # from camera import Camera
    """Video streaming route. Put this in the src attribute of an img tag."""
    response = Response(get_frame(), mimetype="image/jpeg")
    response.headers.add("Access-Control-Allow-Origin", "*")
    return response

@app.route('/mjpg.png')  ##picture
def video_feed_png():
    # from camera import Camera
    """Video streaming route. Put this in the src attribute of an img tag."""
    response = Response(get_png_frame(), mimetype="image/png")
    response.headers.add("Access-Control-Allow-Origin", "*")
    return response



def web_camera_start():
    app.run(host='0.0.0.0', port=9000,threaded=True)

# 滤镜
EFFECTS = [   
    "none",
    "negative",
    "solarize",
    "emboss",
    "posterise",
    "cartoon",
]


Camera_SETTING = [
        "resolution",    #max(4056,3040)
        #"framerate 
        "rotation",      #(0 90 180 270)
        # "shutter_speed",
        "brightness",    # 0~100  default 50
        "sharpness",    # -100~100  default 0
        "contrast",    # -100~100  default 0
        "saturation",    # -100~100  default 0
        "iso",           #Vaild value:0(auto) 100,200,320,400,500,640,800
        "exposure_compensation",       # -25~25  default 0
        "exposure_mode",       #Valid values are: 'off', 'auto' (default),'night', 'nightpreview', 'backlight', 'spotlight', 'sports', 'snow', 'beach','verylong', 'fixedfps', 'antishake', or 'fireworks'
        "meter_mode",     #Valid values are: 'average' (default),'spot', 'backlit', 'matrix'.
        "awb_mode",       #'off', 'auto' (default), ‘sunlight', 'cloudy', 'shade', 'tungsten', 'fluorescent','incandescent', 'flash', or 'horizon'.
        "hflip",          # Default:False ,True
        "vflip",          # Default:False ,True
        # "crop",           #Retrieves or sets the zoom applied to the camera’s input, as a tuple (x, y, w, h) of floating point
                          #values ranging from 0.0 to 1.0, indicating the proportion of the image to include in the output
                          #(the ‘region of interest’). The default value is (0.0, 0.0, 1.0, 1.0), which indicates that everything
                          #should be included.
]

# 相片水印
time_font = lambda x: ImageFont.truetype('/opt/ezblock/Roboto-Light-2.ttf', int(x / 320.0 * 6))
text_font = lambda x: ImageFont.truetype('/opt/ezblock/Roboto-Light-2.ttf', int(x / 320.0 * 10))
company_font = lambda x: ImageFont.truetype('/opt/ezblock/Roboto-Light-2.ttf', int(x / 320.0 * 8))

# 添加水印接口
def add_text_to_image(name, text_1):

    image_target = Image.open(name)

    image_draw = ImageDraw.Draw(image_target)

    
    time_text = time.strftime("%Y-%m-%d %H:%M:%S", time.localtime())
    time_size_x, time_size_y = image_draw.textsize(time_text, font=time_font(image_target.size[0]))
    text_size_x, text_size_y = image_draw.textsize(text_1, font=text_font(image_target.size[0]))

  # 设置文本文字位置
    # print(rgba_image)
    time_xy = (image_target.size[0] - time_size_x - time_size_y, image_target.size[1] - int(1.5*time_size_y))
    text_xy = (text_size_y, image_target.size[1] - int(1.5*text_size_y))
    company_xy = (text_size_y, image_target.size[1] - int(1.5*text_size_y) - text_size_y)

  # 设置文本颜色和透明度
    image_draw.text(time_xy, time_text, font=time_font(image_target.size[0]), fill=(255, 255, 255))
    image_draw.text(company_xy, text_1, font=text_font(image_target.size[0]), fill=(255, 255, 255))
    image_target.save(name,quality=95,subsampling=0)# 


class Vilib(object): 

    video_flag = False

# 读取人脸识别模型
    face_cascade = cv2.CascadeClassifier('/opt/ezblock/haarcascade_frontalface_default.xml') 
    kernel_5 = np.ones((5,5),np.uint8)#4x4的卷积核

    video_source = 0

# 用于寻找手势识别的肤色的区域的模板图片，可以通过手势识别的校准功能更改图片
    roi = cv2.imread("/opt/ezblock/cali.jpg")
    roi_hsv = cv2.cvtColor(roi, cv2.COLOR_BGR2HSV)

# 创建共享字典，提供外部接口动态修改，以及返回字典内容
    detect_obj_parameter = Manager().dict()
    img_array = Manager().list(range(2))

# 默认的颜色识别颜色为红色
    detect_obj_parameter['color_default'] = 'red'

# 颜色的HSV空间中的 H 的范围
    color_dict = {'red':[0,4],'orange':[5,18],'yellow':[22,37],'green':[42,85],'blue':[92,110],'purple':[115,165],'red_2':[165,180]}

# color_obj_parameter
    detect_obj_parameter['color_x'] = 320
    detect_obj_parameter['color_y'] = 240
    detect_obj_parameter['color_w'] = 0
    detect_obj_parameter['color_h'] = 0
    detect_obj_parameter['color_n'] = 0
    detect_obj_parameter['lower_color'] = np.array([min(color_dict[detect_obj_parameter['color_default']]), 60, 60]) 
    detect_obj_parameter['upper_color'] = np.array([max(color_dict[detect_obj_parameter['color_default']]), 255, 255])
    

# Human_obj_parameter
    detect_obj_parameter['human_x'] = 320
    detect_obj_parameter['human_y'] = 240
    detect_obj_parameter['human_w'] = 0
    detect_obj_parameter['human_h'] = 0
    detect_obj_parameter['human_n'] = 0

# traffic_sign_obj_parameter
    detect_obj_parameter['traffic_sign_x'] = 320
    detect_obj_parameter['traffic_sign_y'] = 240
    detect_obj_parameter['traffic_sign_w'] = 0
    detect_obj_parameter['traffic_sign_h'] = 0
    detect_obj_parameter['traffic_sign_t'] = 'None'
    detect_obj_parameter['traffic_sign_acc'] = 0

# gesture_obj_parameter
    detect_obj_parameter['gesture_x'] = 320
    detect_obj_parameter['gesture_y'] = 240
    detect_obj_parameter['gesture_w'] = 0
    detect_obj_parameter['gesture_h'] = 0
    detect_obj_parameter['gesture_t'] = 'None'
    detect_obj_parameter['gesture_acc'] = 0
    # detect_obj_parameter['human_n'] = 0


# detect_switch
    detect_obj_parameter['hdf_flag'] = False
    detect_obj_parameter['cdf_flag'] = False
    detect_obj_parameter['ts_flag'] = False
    detect_obj_parameter['gs_flag'] = False
    detect_obj_parameter['calibrate_flag'] = False   
    detect_obj_parameter['object_follow_flag'] = False
    detect_obj_parameter['qr_flag'] = False

# QR_code
    detect_obj_parameter['qr_data'] = "None"
    detect_obj_parameter['qr_x'] = 320
    detect_obj_parameter['qr_y'] = 240
    detect_obj_parameter['qr_w'] = 0
    detect_obj_parameter['qr_h'] = 0


# picture
    detect_obj_parameter['picture_flag'] = False
    detect_obj_parameter['process_picture'] = True
    detect_obj_parameter['picture_path'] = '/home/pi/picture_file/' + datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')+ '.jpg'

    detect_obj_parameter['video_flag'] = None

    detect_obj_parameter['ensure_flag'] = False
    detect_obj_parameter['clarity_val'] = 0

# diy
    detect_obj_parameter['human_n'] = 0
    # detect_obj_parameter['hdf_flag'] = False

# picture
    detect_obj_parameter['eff'] = 0
    detect_obj_parameter['setting'] = 0
    detect_obj_parameter['setting_flag'] = False
    detect_obj_parameter['setting_val'] = 0
    # detect_obj_parameter['current_setting_val'] = None
    detect_obj_parameter['setting_resolution'] = (3840,2880)
    detect_obj_parameter['change_setting_flag'] = False
    detect_obj_parameter['change_setting_type'] = 'None'
    detect_obj_parameter['change_setting_val'] = 0

    detect_obj_parameter['photo_button_flag'] = False
    detect_obj_parameter['content_length'] = 0
    detect_obj_parameter['content_num'] = 0
    detect_obj_parameter['process_content_1'] = []
    detect_obj_parameter['process_si'] = []
    # detect_obj_parameter['process_dict'] = {}

    detect_obj_parameter['watermark_flag'] = True
    detect_obj_parameter['camera_flip'] = False
    detect_obj_parameter['watermark'] = "Shot by Picar-x"
    # detect_obj_parameter['google_upload_flag'] = False

    rt_img = np.ones((320,240),np.uint8)
    front_view_img = np.zeros((240,320,3), np.uint8)
# 使用白色填充图片区域,默认为黑色
    # front_view_img.fill(255)       
    img_array[0] = rt_img
    img_array[1] = rt_img
    # img_array = rt_img
    vi_img = np.ones((320,240),np.uint8)  


# 通过两个参数Shift_left，Shift_right修改
    @staticmethod
    def photo_effect(shirt_way = 'Shift_left'):
        print(shirt_way)
        shirt_way = str(shirt_way)
        if shirt_way == 'Shift_left':
            Vilib.detect_obj_parameter['eff'] += 1
            if Vilib.detect_obj_parameter['eff'] >= len(EFFECTS):
                Vilib.detect_obj_parameter['eff'] = 0
        elif shirt_way == 'Shift_right':
            Vilib.detect_obj_parameter['eff'] -= 1
            if Vilib.detect_obj_parameter['eff'] < 0:
                Vilib.detect_obj_parameter['eff'] = len(EFFECTS) - 1
        else:
            raise Exception("parameter error!")


    @staticmethod
    def video_flag(flag):
        # global button_motion
        Vilib.detect_obj_parameter['video_flag'] = flag


    @staticmethod
    def watermark(watermark = "Shot by Picar-x"):
        # global button_motion
        watermark = str(watermark)
        Vilib.detect_obj_parameter['watermark_flag'] = True
        Vilib.detect_obj_parameter['watermark'] = watermark

    @staticmethod
    def show_setting(flag):
        # global button_motion

        Vilib.detect_obj_parameter['setting_flag'] = flag
        # button_motion = 'free'

    @staticmethod
    def change_setting_type_val(setting_type,setting_val):
        # global button_motion
        if setting_type == 'resolution':
            Vilib.detect_obj_parameter['setting_resolution'] = setting_val
        else:
            Vilib.detect_obj_parameter['change_setting_type'] = setting_type
            Vilib.detect_obj_parameter['change_setting_val'] = setting_val
            Vilib.detect_obj_parameter['change_setting_flag'] = True


    @staticmethod
    def shuttle_button():
        Vilib.detect_obj_parameter['photo_button_flag']  = True
        

    @staticmethod
    def make_qrcode_picture(data):

        Vilib.img_array = qrcode.make(data=data)


# 返回检测到的颜色的坐标，大小，数量
    @staticmethod
    def color_detect_object(obj_parameter):
        if obj_parameter == 'x':         
            return int(Vilib.detect_obj_parameter['color_x']/214.0)-1
        elif obj_parameter == 'y':
            return -1*(int(Vilib.detect_obj_parameter['color_y']/160.2)-1) #max_size_object_coordinate_y
        elif obj_parameter == 'width':
            return Vilib.detect_obj_parameter['color_w']   #objects_max_width
        elif obj_parameter == 'height':
            return Vilib.detect_obj_parameter['color_h']   #objects_max_height
        elif obj_parameter == 'number':      
            return Vilib.detect_obj_parameter['color_n']   #objects_count
        return None

# 返回检测到的人脸的坐标，大小，数量
    @staticmethod
    def human_detect_object(obj_parameter):
        if obj_parameter == 'x':        
            return int(Vilib.detect_obj_parameter['human_x']/214.0)-1
        elif obj_parameter == 'y':
            return -1*(int(Vilib.detect_obj_parameter['human_y']/160.2)-1) #max_size_object_coordinate_y
        elif obj_parameter == 'width':
            return Vilib.detect_obj_parameter['human_w']   #objects_max_width
        elif obj_parameter == 'height':
            return Vilib.detect_obj_parameter['human_h']   #objects_max_height
        elif obj_parameter == 'number':      
            return Vilib.detect_obj_parameter['human_n']   #objects_count
        return None

# 返回检测到的交通标志的坐标，大小，类型，准确度
    @staticmethod
    def traffic_sign_detect_object(obj_parameter):
        if obj_parameter == 'x':         
            return int(Vilib.detect_obj_parameter['traffic_sign_x']/214.0)-1
        elif obj_parameter == 'y':
            return -1*(int(Vilib.detect_obj_parameter['traffic_sign_y']/160.2)-1) #max_size_object_coordinate_y
        elif obj_parameter == 'width':
            return Vilib.detect_obj_parameter['traffic_sign_w']   #objects_max_width
        elif obj_parameter == 'height':
            return Vilib.detect_obj_parameter['traffic_sign_h']   #objects_max_height

        elif obj_parameter == 'type':      
            return Vilib.detect_obj_parameter['traffic_sign_t']   #objects_type
        elif obj_parameter == 'accuracy':      
            return Vilib.detect_obj_parameter['traffic_sign_acc']   #objects_type
        return 'none'

# 返回检测到的手势的坐标，大小，类型，准确度
    @staticmethod
    def gesture_detect_object(obj_parameter):
        if obj_parameter == 'x':         
            return int(Vilib.detect_obj_parameter['gesture_x']/214.0)-1
        elif obj_parameter == 'y':
            return -1*(int(Vilib.detect_obj_parameter['gesture_y']/160.2)-1) #max_size_object_coordinate_y
        elif obj_parameter == 'width':
            return Vilib.detect_obj_parameter['gesture_w']   #objects_max_width
        elif obj_parameter == 'height':
            return Vilib.detect_obj_parameter['gesture_h']   #objects_max_height
        elif obj_parameter == 'type':      
            return Vilib.detect_obj_parameter['gesture_t']   #objects_type
        elif obj_parameter == 'accuracy':      
            return Vilib.detect_obj_parameter['gesture_acc']   #objects_type
        return 'none'

# 返回检测到的二维码的坐标，大小，类型，准确度
    @staticmethod
    def qrcode_detect_object(obj_parameter = 'data'):
        if obj_parameter == 'x':        
            return int(Vilib.detect_obj_parameter['qr_x']/214.0)-1
        elif obj_parameter == 'y':
            return -1*(int(Vilib.detect_obj_parameter['qr_y']/160.2)-1) #max_size_object_coordinate_y
        elif obj_parameter == 'width':
            return Vilib.detect_obj_parameter['qr_w']   #objects_max_width
        elif obj_parameter == 'height':
            return Vilib.detect_obj_parameter['qr_h']   #objects_max_height
        elif obj_parameter == 'data':      
            return Vilib.detect_obj_parameter['qr_data']   #objects_count
        return 'none'

# 设置要检测的颜色
    @staticmethod
    def detect_color_name(color_name):
        Vilib.detect_obj_parameter['color_default'] = color_name
        Vilib.detect_obj_parameter['lower_color'] = np.array([min(Vilib.color_dict[Vilib.detect_obj_parameter['color_default']]), 60, 60])  
        Vilib.detect_obj_parameter['upper_color'] = np.array([max(Vilib.color_dict[Vilib.detect_obj_parameter['color_default']]), 255, 255])
        Vilib.detect_obj_parameter['cdf_flag']  = True

# 开启摄像头网络传输，web_func参数可以控制是否开启网络传输，不开启网络传输也可以进行识别，两者是不同的线程
    @staticmethod
    def camera_start(web_func = True,inverted_flag = False):

        if inverted_flag == True:
            Vilib.detect_obj_parameter['camera_flip'] = True
        else:
            Vilib.detect_obj_parameter['camera_flip'] = False

        worker_2 = threading.Thread(target=Vilib.camera_clone, name="Thread1")
        if web_func == True: 
            worker_1 = threading.Thread(name='worker 1',target=web_camera_start)
            worker_1.start()
        worker_2.start()


# 人脸检测开关    
    @staticmethod
    def human_detect_switch(flag=False):
        Vilib.detect_obj_parameter['hdf_flag'] = flag

# 颜色检测开关
    @staticmethod
    def color_detect_switch(flag=False):
        Vilib.detect_obj_parameter['cdf_flag']  = flag

# 手势检测开关
    @staticmethod
    def gesture_detect_switch(flag=False):
        Vilib.detect_obj_parameter['gs_flag']  = flag

# 交通标志检测开关
    @staticmethod
    def traffic_sign_detect_switch(flag=False):
        Vilib.detect_obj_parameter['ts_flag']  = flag

# 手势检测开关
    @staticmethod
    def gesture_calibrate_switch(flag=False):
        Vilib.detect_obj_parameter['calibrate_flag']  = flag

# 目标检测开关
    @staticmethod
    def object_follow_switch(flag=False):
        Vilib.detect_obj_parameter['object_follow_flag'] = flag

# 二维码检测开关
    @staticmethod
    def qrcode_detect_switch(flag=False):
        Vilib.detect_obj_parameter['qr_flag']  = flag

# 摄像头的线程
    @staticmethod
    def camera_clone():
        Vilib.camera()     

    @staticmethod
    def camera():
        global effect
        camera = PiCamera()
        camera.resolution = (640, 480)
        camera.image_effect = EFFECTS[Vilib.detect_obj_parameter['eff']]
        camera.framerate = 24
        camera.rotation = 0
        # camera.rotation = 180   
        camera.brightness = 50    #(0 to 100)
        camera.sharpness = 0      #(-100 to 100)
        camera.contrast = 0       #(-100 to 100)
        camera.saturation = 0     #(-100 to 100)
        camera.iso = 0            #(automatic)(100 to 800)
        camera.exposure_compensation = 0   #(-25 to 25)
        camera.exposure_mode = 'auto'
        camera.meter_mode = 'average'
        camera.awb_mode = 'auto'
        camera.hflip = False
        camera.vflip = Vilib.detect_obj_parameter['camera_flip']
        camera.crop = (0.0, 0.0, 1.0, 1.0)
        rawCapture = PiRGBArray(camera, size=camera.resolution)  
        last_e ='none'
        camera_val = 0
        last_show_content_list = []
        show_content_list = []
        change_type_val  = []
        change_type_dict = {"shutter_speed":0,"resolution":[2592,1944], "brightness":50, "contrast":0, "sharpness":0, "saturation":0, "iso":0, "exposure_compensation":0, "exposure_mode":'auto', \
            "meter_mode":'average' ,"rotation":0 ,"awb_mode":'auto',"drc_strength":'off',"hflip":False,"vflip":True}
        start_time = 0
        end_time = 0
        # camera.framerate = 10
        # 
        try:
            while True:


                for frame in camera.capture_continuous(rawCapture, format="bgr",use_video_port=True):# use_video_port=True
                    
                    start_time = time.time()
                    img = frame.array

                    img = Vilib.gesture_calibrate(img)
                    img = Vilib.traffic_detect(img)
                    img = Vilib.color_detect_func(img)
                    img = Vilib.human_detect_func(img)
                    img = Vilib.gesture_recognition(img)
                    img = Vilib.qrcode_detect_func(img)
                    
                    # change_camera_setting
                    if Vilib.detect_obj_parameter['change_setting_flag'] == True:
                        Vilib.detect_obj_parameter['change_setting_flag'] = False

                        change_setting_cmd = "camera." + Vilib.detect_obj_parameter['change_setting_type'] + '=' + str(Vilib.detect_obj_parameter['change_setting_val'])
                        print(change_setting_cmd)
                        exec(change_setting_cmd)

                        change_type_dict[Vilib.detect_obj_parameter['change_setting_type']] = Vilib.detect_obj_parameter['change_setting_val']
                    if Vilib.detect_obj_parameter['content_num'] != 0:

                        for i in range(Vilib.detect_obj_parameter['content_num']):
                            exec("Vilib.detect_obj_parameter['process_si'] = Vilib.detect_obj_parameter['process_content_" + str(i+1) + "'" + "]")
                            cv2.putText(img, str(Vilib.detect_obj_parameter['process_si'][0]),Vilib.detect_obj_parameter['process_si'][1],cv2.FONT_HERSHEY_SIMPLEX,Vilib.detect_obj_parameter['process_si'][3],Vilib.detect_obj_parameter['process_si'][2],2)
                    
                    if Vilib.detect_obj_parameter['setting_flag'] == True:
                        setting_type = Camera_SETTING[Vilib.detect_obj_parameter['setting']]
                        if setting_type == "resolution":
                            Vilib.detect_obj_parameter['setting_val'] = Vilib.detect_obj_parameter['setting_resolution']

                            change_type_dict["resolution"] = list(Vilib.detect_obj_parameter['setting_resolution'])
                            cv2.putText(img, 'resolution:' + str(Vilib.detect_obj_parameter['setting_resolution']),(10,20),cv2.FONT_HERSHEY_SIMPLEX,0.6,(255,255,255),2)
                        elif setting_type == "shutter_speed":
                            change_type_dict["shutter_speed"] = Vilib.detect_obj_parameter['change_setting_val']
                            cv2.putText(img, 'shutter_speed:' + str(Vilib.detect_obj_parameter['change_setting_val']),(10,20),cv2.FONT_HERSHEY_SIMPLEX,0.6,(255,255,255),2)
                        else:
                            cmd_text = "Vilib.detect_obj_parameter['setting_val'] = camera." + Camera_SETTING[Vilib.detect_obj_parameter['setting']]
                            # print('mennu:',Ras_Cam.detect_obj_parameter['setting_val'])
                            exec(cmd_text)
                            cv2.putText(img, setting_type + ': ' + str(Vilib.detect_obj_parameter['setting_val']),(10,20),cv2.FONT_HERSHEY_SIMPLEX,0.6,(255,255,255),2)


                    e = EFFECTS[Vilib.detect_obj_parameter['eff']]
                    
                    
                    if last_e != e:
                        camera.image_effect = e
                    last_e = e
                    if last_e != 'none':
                        cv2.putText(img, str(last_e),(0,15),cv2.FONT_HERSHEY_SIMPLEX,0.6,(204,209,72),2)

                        
                    if Vilib.detect_obj_parameter['photo_button_flag'] == True:
                        camera.close()
                        break
                            
    
                    Vilib.img_array[0] = img
                    rawCapture.truncate(0)
                    end_time = time.time()
                    end_time = end_time - start_time


                picture_time = datetime.datetime.now().strftime('%Y-%m-%d_%H-%M-%S')
                Vilib.detect_obj_parameter['picture_path'] = '/home/pi/picture_file/' + picture_time + '.jpg'

                a_t = "sudo raspistill -t 250  -w 2592 -h 1944 -vf" + " -rot " + str(change_type_dict['rotation']) + " -ifx " + str(EFFECTS[Vilib.detect_obj_parameter['eff']]) +" -o " + Vilib.detect_obj_parameter['picture_path']
                

                print(a_t)
                run_command(a_t)

                if Vilib.detect_obj_parameter['watermark_flag'] == True:
                    add_text_to_image(Vilib.detect_obj_parameter['picture_path'],Vilib.detect_obj_parameter['watermark'])

                #init again

                camera = PiCamera()
                camera.resolution = (640,480)
                camera.vflip = Vilib.detect_obj_parameter['camera_flip']
                # camera.rotation = Vilib.detect_obj_parameter['camera_rot']
                camera.image_effect = e
                rawCapture = PiRGBArray(camera, size=camera.resolution) 
                Vilib.detect_obj_parameter['photo_button_flag'] = False
                   
        finally:
            camera.close()


# 手势校准接口
    @staticmethod
    def gesture_calibrate(img):
        if Vilib.detect_obj_parameter['calibrate_flag'] == True:
            cv2.imwrite('/opt/ezblock/cali.jpg', img[190:290,270:370])
            cv2.rectangle(img,(270,190),(370,290),(255,255,255),2)

        return img


# 添加水印的控制开关
    @staticmethod
    def get_picture(process_picture):
        Vilib.detect_obj_parameter['picture_flag'] = True
        Vilib.detect_obj_parameter['process_picture'] = process_picture
        Vilib.detect_obj_parameter['picture_path'] = '/home/pi/picture_file/' + datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S') + '.jpg'

# 拍照控制接口
    @staticmethod
    def take_photo(img):
        if img is not None:
            cv2.imwrite(Vilib.detect_obj_parameter['picture_path'], img)
            Vilib.detect_obj_parameter['picture_flag'] = False


    @staticmethod
    def cnt_area(cnt):
        x,y,w,h = cv2.boundingRect(cnt)
        return w*h


# 交通标志检测函数，传入值依此是摄像头读取到图像，交通标志的坐标，长宽
    @staticmethod
    def traffic_predict(input_img,x,y,w,h):

        x1 = int(x)
        x2 = int(x + w)
        y1 = int(y)
        y2 = int(y + h)

        new_img = input_img[y1:y2,x1:x2]
        new_img = (new_img / 255.0)   #归一化
        new_img = (new_img - 0.5) * 2.0  

        resize_img = cv2.resize(new_img, (96,96), interpolation=cv2.INTER_LINEAR)   #调整为识别模型的要求的96x96的图像大小
        flatten_img = np.reshape(resize_img, (96,96,3))
        im5 = np.expand_dims(flatten_img,axis = 0)

    # Perform the actual detection by running the model with the image as input
        image_np_expanded = im5.astype('float32') # 类型也要满足要求

        interpreter_1.set_tensor(input_details_2[0]['index'],image_np_expanded)  #放入图像到模型中
        interpreter_1.invoke()        #检测
        output_data_2 = interpreter_1.get_tensor(output_details_2[0]['index'])   #获取模型返回的数据

    #     # 出来的结果去掉没用的维度   np.where(result==np.max(result)))[0][0]
        result = np.squeeze(output_data_2)
        result_accuracy =  round(np.max(result),2)     #获取准确度
        ges_class = np.where(result==np.max(result))[0][0]   #获取类型


        return result_accuracy,ges_class


### 手势识别的流程和上面交通标志一致
    @staticmethod
    def gesture_predict(input_img,x,y,w,h):

        x1 = int(x)
        x2 = int(x + w)
        y1 = int(y)
        y2 = int(y + h)

        if x1 <= 0:
            x1 = 0
        elif x2 >= 640:
            x2 = 640
        if y1 <= 0:
            y1 = 0
        elif y2 >= 640:
            y2 = 640


        new_img = input_img[y1:y2,x1:x2]
        new_img = (new_img / 255.0)
        new_img = (new_img - 0.5) * 2.0

        resize_img = cv2.resize(new_img, (96,96), interpolation=cv2.INTER_LINEAR)
        flatten_img = np.reshape(resize_img, (96,96,3))
        im5 = np.expand_dims(flatten_img,axis = 0)

    # Perform the actual detection by running the model with the image as input
        image_np_expanded = im5.astype('float32') # 类型也要满足要求

        interpreter_2.set_tensor(input_details_2[0]['index'],image_np_expanded)
        interpreter_2.invoke()
        output_data_2 = interpreter_2.get_tensor(output_details_2[0]['index'])

    #     # 出来的结果去掉没用的维度   np.where(result==np.max(result)))[0][0]
        result = np.squeeze(output_data_2)
        result_accuracy =  round(np.max(result),2)
        ges_class = np.where(result==np.max(result))[0][0]


        return result_accuracy,ges_class


# 交通标志可能存在区域的检测
    @staticmethod
    def traffic_detect(img):

        if Vilib.detect_obj_parameter['ts_flag']  == True:

            hsv = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)              # 2.从BGR转换到HSV
            cv2.circle(img, (160,120), 1, (255,255,255), -1)

            
            ### red
            mask_red_1 = cv2.inRange(hsv,(157,20,20), (180,255,255))
            mask_red_2 = cv2.inRange(hsv,(0,20,20), (10,255,255))

            ### blue
            mask_blue = cv2.inRange(hsv,(92,10,10), (125,255,255))

            ### all
            mask_all = cv2.bitwise_or(mask_red_1, mask_blue)
            
            mask_all = cv2.bitwise_or(mask_red_2, mask_all)
            

            open_img = cv2.morphologyEx(mask_all, cv2.MORPH_OPEN,Vilib.kernel_5,iterations=1)              #开运算 

            contours, hierarchy = cv2.findContours(open_img,cv2.RETR_EXTERNAL,cv2.CHAIN_APPROX_SIMPLE)          ####在binary中发现轮廓，轮廓按照面积从小到大排列
                # p=0
            contours = sorted(contours,key = Vilib.cnt_area, reverse=False)
            traffic_n = len(contours)
            max_area = 0
            traffic_sign_num = 0

            if traffic_n > 0: 
                for i in contours:    # 遍历所有的轮廓
                    x,y,w,h = cv2.boundingRect(i)      # 将轮廓分解为识别对象的左上角坐标和宽、高

                        # 在图像上画上矩形（图片、左上角坐标、右下角坐标、颜色、线条宽度）
                    if w > 32 and h > 32: 

                        acc_val, traffic_type = Vilib.traffic_predict(img,x,y,w,h)
                        # print(traffic_type,acc_val)
                        acc_val = round(acc_val*100)
                        if acc_val >= 75:   

                            if traffic_type == 1 or traffic_type == 2 or traffic_type == 3:


                                simple_gray = cv2.cvtColor(img[y:y+h,x:x+w], cv2.COLOR_BGR2GRAY)
                                # new_mask_blue = cv2.inRange(hsv[y:y+h,x:x+w],(92,70,50), (118,255,255))
                                circles = cv2.HoughCircles(simple_gray,cv2.HOUGH_GRADIENT,1,32,\
                                param1=140,param2=70,minRadius=int(w/4.0),maxRadius=max(w,h))
                               
                                if circles is not None:
                                    for i in circles[0,:]:
                                    # cv2.rectangle(img,(x,y),(x+w,y+h),(0,255,0),2) 
                                        traffic_sign_coor = (int(x+i[0]),int(y+i[1]))
                                        cv2.circle(img,traffic_sign_coor,i[2],(255,0,255),2)
                                        cv2.putText(img,str(traffic_dict[traffic_type]) +': ' + str(round(acc_val)),(int(x+i[0]-i[2]),int(y+i[1]-i[2])), cv2.FONT_HERSHEY_SIMPLEX, 1,(255,0,255),2)#加减10是调整字符位置
                                        if w * h > max_area:
                                            max_area = w * h
                                            max_obj_x = x
                                            max_obj_y = y
                                            max_obj_w = w
                                            max_obj_h = h
                                            max_obj_t = traffic_type
                                            max_obj_acc = acc_val
                                            traffic_sign_num += 1

                            elif traffic_type == 0:
                                # small_hsv = cv2.cvtColor(resize_img, cv2.COLOR_BGR2HSV)
                                red_mask_1 = cv2.inRange(hsv[y:y+h,x:x+w],(0,50,20), (4,255,255))           # 3.inRange()：介于lower/upper之间的为白色，其余黑色
                                red_mask_2 = cv2.inRange(hsv[y:y+h,x:x+w],(163,50,20), (180,255,255))
                                red_mask_all = cv2.bitwise_or(red_mask_1,red_mask_2)

                                        
                                # circles = np.uint16(np.around(circles))

                                # ret, new_binary = cv2.threshold(simple_gray, 0, 255, cv2.THRESH_BINARY_INV | cv2.THRESH_OTSU)
                                new_binary = cv2.GaussianBlur(red_mask_all, (5, 5), 0)

                                open_img = cv2.morphologyEx(red_mask_all, cv2.MORPH_OPEN,Vilib.kernel_5,iterations=1)              #开运算  
                                open_img = cv2.dilate(open_img, Vilib.kernel_5,iterations=5) 

                                blue_contours, hierarchy = cv2.findContours(open_img,cv2.RETR_EXTERNAL,cv2.CHAIN_APPROX_SIMPLE)          ####在binary中发现轮廓，轮廓按照面积从小到大排列
                                
                                contours_count = len(blue_contours)
                                if contours_count >=1:
                                # print("contours:",contours_count)
                                    blue_contours = sorted(blue_contours,key = Vilib.cnt_area, reverse=True)
                                
                                
                                    epsilon = 0.025 * cv2.arcLength(blue_contours[0], True)
                                    approx = cv2.approxPolyDP(blue_contours[0], epsilon, True)

                                #     # 分析几何形状
                                    corners = len(approx)
                                    

                                    if corners >= 0:
                                        traffic_sign_coor = (int(x+w/2),int(y+h/2))
                                        cv2.rectangle(img,(x,y),(x+w,y+h),(255,0,255),2)
                                        cv2.putText(img,str(traffic_dict[traffic_type]) +': ' + str(round(acc_val)),(x,y), cv2.FONT_HERSHEY_SIMPLEX, 1,(255,0,255),2)#加减10是调整字符位置
                                        if w * h > max_area:
                                            max_area = w * h
                                            max_obj_x = x
                                            max_obj_y = y
                                            max_obj_w = w
                                            max_obj_h = h
                                            max_obj_t = traffic_type
                                            max_obj_acc = acc_val
                                            traffic_sign_num += 1

                                        
                # print("traffic_sign_num:",traffic_sign_num)         
                if traffic_sign_num > 0:

                    Vilib.detect_obj_parameter['traffic_sign_x'] = int(max_obj_x + max_obj_w/2)
                    Vilib.detect_obj_parameter['traffic_sign_y'] = int(max_obj_y + max_obj_h/2)
                    Vilib.detect_obj_parameter['traffic_sign_w'] = max_obj_w
                    Vilib.detect_obj_parameter['traffic_sign_h'] = max_obj_h
                    # print("traffic_sign_type:",)
                    Vilib.detect_obj_parameter['traffic_sign_t'] = traffic_dict[max_obj_t]
                    Vilib.detect_obj_parameter['traffic_sign_acc'] = max_obj_acc
                else:
                    Vilib.detect_obj_parameter['traffic_sign_x'] = 320
                    Vilib.detect_obj_parameter['traffic_sign_y'] = 240
                    Vilib.detect_obj_parameter['traffic_sign_w'] = 0
                    Vilib.detect_obj_parameter['traffic_sign_h'] = 0
                    Vilib.detect_obj_parameter['traffic_sign_t'] = 'none'
                    Vilib.detect_obj_parameter['traffic_sign_acc'] = 0

        else:
            Vilib.detect_obj_parameter['traffic_sign_x'] = 320
            Vilib.detect_obj_parameter['traffic_sign_y'] = 240
            Vilib.detect_obj_parameter['traffic_sign_w'] = 0
            Vilib.detect_obj_parameter['traffic_sign_h'] = 0
            Vilib.detect_obj_parameter['traffic_sign_t'] = 'none'
            Vilib.detect_obj_parameter['traffic_sign_acc'] = 0

        return img


# 手掌肤色的区域检测，把图像的区域给手势识别接口做手势识别
    @staticmethod
    def gesture_recognition(img):
        if Vilib.detect_obj_parameter['gs_flag'] == True:

    ###肤色部分

            target_hsv = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)
            # 首先对样本图像计算2D直方图
            roi_hsv_hist = cv2.calcHist([Vilib.roi_hsv], [0, 1], None, [180, 256], [0, 180, 0, 255])
            # 对得到的样本2D直方图进行归一化
            # 这样可以方便显示，归一化后的直方图就变成0-255之间的数了
            # cv2.NORM_MINMAX表示对数组所有值进行转换，线性映射到最大最小值之间
            cv2.normalize(roi_hsv_hist, roi_hsv_hist, 0, 255, cv2.NORM_MINMAX)
            # 对待检测图像进行反向投影
            # 最后一个参数为尺度参数
            dst = cv2.calcBackProject([target_hsv], [0, 1], roi_hsv_hist, [0, 180, 0, 256], 1)
            # 构建一个圆形卷积核，用于对图像进行平滑，连接分散的像素
            disc = cv2.getStructuringElement(cv2.MORPH_ELLIPSE, (5, 5))
            dst = cv2.filter2D(dst, -1, disc,dst)
            ret, thresh = cv2.threshold(dst, 1, 255, 0)
            dilate = cv2.dilate(thresh, Vilib.kernel_5, iterations=3)
                # 注意由于原图是三通道BGR图像，因此在进行位运算之前，先要把thresh转成三通道
            # thresh = cv2.merge((dilate, dilate, dilate))
                # 对原图与二值化后的阈值图像进行位运算，得到结果
            # res = cv2.bitwise_and(img, thresh)
            
            # ycrcb=cv2.cvtColor(img,cv2.COLOR_BGR2YCR_CB)

            # cr_skin = cv2.inRange(ycrcb, (85,124,121), (111,131,128))

            # open_img = cv2.morphologyEx(cr_skin, cv2.MORPH_OPEN,Vilib.kernel_5,iterations=1)

            contours, hierarchy = cv2.findContours(dilate,cv2.RETR_EXTERNAL,cv2.CHAIN_APPROX_SIMPLE)
            ges_num = len(contours)


            if ges_num > 0:
                contours = sorted(contours,key = Vilib.cnt_area, reverse=True)
                # for i in range(0,len(contours)):    #遍历所有的轮廓
                x,y,w,h = cv2.boundingRect(contours[0])      #将轮廓分解为识别对象的左上角坐标和宽、高
                gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY) 
                faces = Vilib.face_cascade.detectMultiScale(gray[y:y+h,x:x+w], 1.3, 2)
            # print(len(faces))
                face_len = len(faces)
                    

                        #在图像上画上矩形（图片、左上角坐标、右下角坐标、颜色、线条宽度）

                    
                if w >= 60 and h >= 60 and face_len == 0:
                    # acc_val,ges_type = Vilib.gesture_predict(img,x-2.2*w,y-2.8*h,4.4*w,5.6*h) 
                    acc_val,ges_type = Vilib.gesture_predict(img,x-0.1*w,y-0.2*h,1.1*w,1.2*h) 

                    acc_val = round(acc_val*100,3)
                    if acc_val >= 75:
                        # print(x,y,w,h)
                        cv2.rectangle(img,(int(x-0.1*w),int(y-0.2*h)),(int(x+1.1*w), int(y+1.2*h)),(0,125,0),2, cv2.LINE_AA)
                        cv2.rectangle(img,(0,0),(125,27),(204,209,72),-1, cv2.LINE_AA)
                        cv2.putText(img,ges_dict[ges_type]+': '+str(acc_val) + '%',(0,17),cv2.FONT_HERSHEY_SIMPLEX,0.6,(255,255,255),2)  ##(0,97,240)


                        Vilib.detect_obj_parameter['gesture_x'] = int(x + w/2)
                        Vilib.detect_obj_parameter['gesture_y'] = int(y + h/2)
                        Vilib.detect_obj_parameter['gesture_w'] = w
                        Vilib.detect_obj_parameter['gesture_h'] = h
                        Vilib.detect_obj_parameter['gesture_t'] = ges_dict[ges_type]
                        Vilib.detect_obj_parameter['gesture_acc'] = acc_val
                                # print()
                    else:
                        Vilib.detect_obj_parameter['gesture_x'] = 320
                        Vilib.detect_obj_parameter['gesture_y'] = 240
                        Vilib.detect_obj_parameter['gesture_w'] = 0
                        Vilib.detect_obj_parameter['gesture_h'] = 0
                        Vilib.detect_obj_parameter['gesture_t'] = 'none'
                        Vilib.detect_obj_parameter['gesture_acc'] = 0


                else:
                    Vilib.detect_obj_parameter['gesture_x'] = 320
                    Vilib.detect_obj_parameter['gesture_y'] = 240
                    Vilib.detect_obj_parameter['gesture_w'] = 0
                    Vilib.detect_obj_parameter['gesture_h'] = 0
                    Vilib.detect_obj_parameter['gesture_t'] = 'none'
                    Vilib.detect_obj_parameter['gesture_acc'] = 0

            else:
                Vilib.detect_obj_parameter['gesture_x'] = 320
                Vilib.detect_obj_parameter['gesture_y'] = 240
                Vilib.detect_obj_parameter['gesture_w'] = 0
                Vilib.detect_obj_parameter['gesture_h'] = 0
                Vilib.detect_obj_parameter['gesture_t'] = 'none'
                Vilib.detect_obj_parameter['gesture_acc'] = 0

        return img


# 人脸检测
    @staticmethod
    def human_detect_func(img):
        if Vilib.detect_obj_parameter['hdf_flag'] == True:
            resize_img = cv2.resize(img, (320,240), interpolation=cv2.INTER_LINEAR)            # 2.从BGR转换到RAY
            gray = cv2.cvtColor(resize_img, cv2.COLOR_BGR2GRAY) 
            faces = Vilib.face_cascade.detectMultiScale(gray, 1.3, 2)
            # print(len(faces))
            Vilib.detect_obj_parameter['human_n'] = len(faces)
            max_area = 0
            if Vilib.detect_obj_parameter['human_n'] > 0:
                for (x,y,w,h) in faces:
                    
                    x = x*2
                    y = y*2
                    w = w*2
                    h = h*2

                    cv2.rectangle(img,(x,y),(x+w,y+h),(255,0,0),2)
                    object_area = w*h
                    if object_area > max_area: 
                        object_area = max_area
                        Vilib.detect_obj_parameter['human_x'] = int(x + w/2)
                        Vilib.detect_obj_parameter['human_y'] = int(y + h/2)
                        Vilib.detect_obj_parameter['human_w'] = w
                        Vilib.detect_obj_parameter['human_h'] = h
            
            else:
                Vilib.detect_obj_parameter['human_x'] = 320
                Vilib.detect_obj_parameter['human_y'] = 240
                Vilib.detect_obj_parameter['human_w'] = 0
                Vilib.detect_obj_parameter['human_h'] = 0
                Vilib.detect_obj_parameter['human_n'] = 0
            return img
        else:
            return img




# 颜色识别

    @staticmethod
    def color_detect_func(img):

        # 蓝色的范围，不同光照条件下不一样，可灵活调整   H：色度，S：饱和度 v:明度
        if Vilib.detect_obj_parameter['cdf_flag']  == True:
            resize_img = cv2.resize(img, (160,120), interpolation=cv2.INTER_LINEAR)
            hsv = cv2.cvtColor(resize_img, cv2.COLOR_BGR2HSV)              # 2.从BGR转换到HSV
            # print(Vilib.lower_color)
            color_type = Vilib.detect_obj_parameter['color_default']
            
            mask = cv2.inRange(hsv,np.array([min(Vilib.color_dict[color_type]), 60, 60]), np.array([max(Vilib.color_dict[color_type]), 255, 255]) )           # 3.inRange()：介于lower/upper之间的为白色，其余黑色
            if color_type == 'red':
                 mask_2 = cv2.inRange(hsv, (167,0,0), (180,255,255))
                 mask = cv2.bitwise_or(mask, mask_2)

            open_img = cv2.morphologyEx(mask, cv2.MORPH_OPEN,Vilib.kernel_5,iterations=1)              #开运算  

            contours, hierarchy = cv2.findContours(open_img,cv2.RETR_EXTERNAL,cv2.CHAIN_APPROX_SIMPLE)          ####在binary中发现轮廓，轮廓按照面积从小到大排列
                # p=0
            Vilib.detect_obj_parameter['color_n'] = len(contours)
            max_area = 0

            if Vilib.detect_obj_parameter['color_n'] > 0: 
                for i in contours:    #遍历所有的轮廓
                    x,y,w,h = cv2.boundingRect(i)      #将轮廓分解为识别对象的左上角坐标和宽、高

                        #在图像上画上矩形（图片、左上角坐标、右下角坐标、颜色、线条宽度）
                    if w >= 8 and h >= 8: 
                        x = x*4
                        y = y*4
                        w = w*4
                        h = h*4
                        cv2.rectangle(img,(x,y),(x+w,y+h),(0,255,0),2)
                                #给识别对象写上标号
                        cv2.putText(img,color_type,(x,y), cv2.FONT_HERSHEY_SIMPLEX, 1,(0,0,255),2)#加减10是调整字符位置
 
                        object_area = w*h
                        if object_area > max_area: 
                            max_area = object_area
                            Vilib.detect_obj_parameter['color_x'] = int(x + w/2)
                            Vilib.detect_obj_parameter['color_y'] = int(y + h/2)
                            Vilib.detect_obj_parameter['color_w'] = w
                            Vilib.detect_obj_parameter['color_h'] = h
                            # print()
            else:
                Vilib.detect_obj_parameter['color_x'] = 320
                Vilib.detect_obj_parameter['color_y'] = 240
                Vilib.detect_obj_parameter['color_w'] = 0
                Vilib.detect_obj_parameter['color_h'] = 0
                Vilib.detect_obj_parameter['color_n'] = 0
            return img
        else:
            return img

# 二维码识别
    @staticmethod
    def qrcode_detect_func(img):
        if Vilib.detect_obj_parameter['qr_flag']  == True:
            barcodes = pyzbar.decode(img)
            # 循环检测到的条形码
            if len(barcodes) > 0:
                for barcode in barcodes:
                    # 提取条形码的边界框的位置
                    # 画出图像中条形码的边界框
                    (x, y, w, h) = barcode.rect
                    cv2.rectangle(img, (x, y), (x + w, y + h), (0, 0, 255), 2)

                    # 条形码数据为字节对象，所以如果我们想在输出图像上
                    # 画出来，就需要先将它转换成字符串
                    barcodeData = barcode.data.decode("utf-8")
                    # barcodeType = barcode.type

                    # 绘出图像上条形码的数据和条形码类型
                    # text = "{} ({})".format(barcodeData, barcodeType)
                    text = "{}".format(barcodeData)
                    if len(text) > 0:
                        Vilib.detect_obj_parameter['qr_data'] = text
                        Vilib.detect_obj_parameter['qr_h'] = h
                        Vilib.detect_obj_parameter['qr_w'] = w
                        Vilib.detect_obj_parameter['qr_x'] = x 
                        Vilib.detect_obj_parameter['qr_y'] = y
                    # print("Vilib.qr_date:%s"%Vilib.qr_date)
                    cv2.putText(img, text, (x - 20, y - 10), cv2.FONT_HERSHEY_SIMPLEX,
                                0.5, (0, 0, 255), 2)
            else:
                Vilib.detect_obj_parameter['qr_data'] = "None"
                Vilib.detect_obj_parameter['qr_x'] = 320
                Vilib.detect_obj_parameter['qr_y'] = 240
                Vilib.detect_obj_parameter['qr_w'] = 0
                Vilib.detect_obj_parameter['qr_h'] = 0
            return img
        else:
            return img

# 
    @staticmethod
    def new_color_detect_func(img,color):
        Vilib.detect_color_name(color)

        # 蓝色的范围，不同光照条件下不一样，可灵活调整   H：色度，S：饱和度 v:明度
        if Vilib.detect_obj_parameter['cdf_flag']  == True:
            resize_img = cv2.resize(img, (160,120), interpolation=cv2.INTER_LINEAR)
            hsv = cv2.cvtColor(resize_img, cv2.COLOR_BGR2HSV)              # 2.从BGR转换到HSV
            # print(Vilib.lower_color)
            color_type = Vilib.detect_obj_parameter['color_default']
            
            mask = cv2.inRange(hsv,np.array([min(Vilib.color_dict[color_type]), 60, 60]), np.array([max(Vilib.color_dict[color_type]), 255, 255]) )           # 3.inRange()：介于lower/upper之间的为白色，其余黑色
            if color_type == 'red':
                 mask_2 = cv2.inRange(hsv, (167,0,0), (180,255,255))
                 mask = cv2.bitwise_or(mask, mask_2)

            open_img = cv2.morphologyEx(mask, cv2.MORPH_OPEN,Vilib.kernel_5,iterations=1)              #开运算  

            contours, hierarchy = cv2.findContours(open_img,cv2.RETR_EXTERNAL,cv2.CHAIN_APPROX_SIMPLE)          ####在binary中发现轮廓，轮廓按照面积从小到大排列
                # p=0
            Vilib.detect_obj_parameter['color_n'] = len(contours)
            max_area = 0

            if Vilib.detect_obj_parameter['color_n'] > 0: 
                for i in contours:    #遍历所有的轮廓
                    x,y,w,h = cv2.boundingRect(i)      #将轮廓分解为识别对象的左上角坐标和宽、高

                        #在图像上画上矩形（图片、左上角坐标、右下角坐标、颜色、线条宽度）
                    if w >= 8 and h >= 8: 
                        x = x*2
                        y = y*2
                        w = w*2
                        h = h*2
                        cv2.rectangle(img,(x,y),(x+w,y+h),(0,255,0),2)
                                #给识别对象写上标号
                        cv2.putText(img,color_type,(x,y), cv2.FONT_HERSHEY_SIMPLEX, 1,(0,0,255),2)#加减10是调整字符位置
 
                        object_area = w*h
                        if object_area > max_area: 
                            max_area = object_area
                            Vilib.detect_obj_parameter['color_x'] = int(x + w/2)
                            Vilib.detect_obj_parameter['color_y'] = int(y + h/2)
                            Vilib.detect_obj_parameter['color_w'] = w
                            Vilib.detect_obj_parameter['color_h'] = h
                            # print()
            else:
                Vilib.detect_obj_parameter['color_x'] = 320
                Vilib.detect_obj_parameter['color_y'] = 240
                Vilib.detect_obj_parameter['color_w'] = 0
                Vilib.detect_obj_parameter['color_h'] = 0
                Vilib.detect_obj_parameter['color_n'] = 0
            return img
        else:
            return img





# 新增 2021.07.30
# 显示在树莓派桌面，在浏览器输入蜘蛛的IP地址可以看到画面
def display()


def take_photo(photo_name,path=‘home\pi\picture’) 

def recode_video(video_name)

def color_detect(color="red")

def face_detect()

def face_training(name,path=‘home\pi\picture’) # 把路径里面的照片训练成名为name模型

def face_recognition() # 返回匹配模型的name

def qr_coder_reader()

if __name__ == "__main__":
    Vilib.camera_start()
    while True:
        pass