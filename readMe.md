# ORBIT ROS 2 Projesinde WebSocket Entegrasyonu İçin Mesaj Tipleri ve Çözümleme Yöntemleri

Bu doküman, ORBIT ROS 2 projesinde kullanılan mesaj tiplerini ve bu mesajların WebSocket üzerinden iletimi sırasında kullanılan çözümleme yöntemlerini açıklamaktadır. Tüm mesajlar, platformdan bağımsız uyumluluk ve kolay entegrasyon sağlamak amacıyla JSON formatında iletilmektedir. Her bir mesaj için ilgili ROS 2 mesaj tipi ve konu (topic) adı açıkça belirtilmiştir; bu sayede istemci tarafında güvenilir veri ayrıştırma ve gerçek zamanlı işlem yapılması mümkün hale gelmektedir.

## Publishers
1. **Map Topic**
   
   Topic: `map_downsampled`

   Msg_type: `amr_websocket_interfaces/CompressedOccupancyGrid`

   Msg_definition:
   ```json
    {
        "map_grid": "nav_msgs/OccupancyGrid",
        "length": "int32[]",
    }
   ```
   [nav_msgs/OccupancyGrid.msg](https://docs.ros.org/en/noetic/api/nav_msgs/html/msg/OccupancyGrid.html)

    **Harita Mesajı Sıkıştırma Yöntemi**

    Harita mesajı, Run-Length Encoding (RLE) algoritması kullanılarak sıkıştırılmıştır. OccupancyGrid mesajı içerisinde, `data` alanı benzersiz değerlerin sırasını içerirken, `length` alanı her bir değerin tekrar sayısını belirtir. Başka bir deyişle, sıkıştırılmış veri şu şekilde açılır:

    `açılmış_veri = data[i] değeri length[i] kadar tekrar edilir`

    Bu yöntem, iletilen verinin boyutunu önemli ölçüde azaltırken, alıcı tarafta tam harita bilgisinin yeniden oluşturulmasını sağlar.

2. **Map Update Topic**
   
   Topic: `updated_map`

   Msg_type: `amr_websocket_interfaces/UpdatedOccupancyGrid`

   Msg_definition:

   ```json
    {
        "header": "std_msgs/Header",
        "index": "int32[]",
        "data": "int8[]"
    }
   ```

   [std_msgs/Header](https://docs.ros2.org/foxy/api/std_msgs/msg/Header.html)

    **Değişen Harita Bölgeleri Konu Açıklaması**

    Bu konu (topic), harita üzerinde değişiklik yapılan bölgeleri yayınlar. `index` alanı, değişen hücrelerin indekslerini içeren bir listedir; `data` alanı ise bu hücrelere karşılık gelen doluluk (occupancy) değerlerini taşır. Bu seçici güncelleme mekanizması, tüm harita verisini yeniden iletmeden yalnızca değişen kısımların verimli bir şekilde iletilmesini sağlar.

3. **Path Topic**
   
   Topic: `websocket_path`

   Msg_type: `amr_websocket_interfaces/WebsocketPath`

   Msg_definition:

   ```json
    {
        "header": "std_msgs/Header",
        "poses": "geometry_msgs/Point[]"
    }
   ```

   [std_msgs/Header](https://docs.ros2.org/foxy/api/std_msgs/msg/Header.html)

   [geometry_msgs/Point](https://docs.ros.org/en/noetic/api/geometry_msgs/html/msg/Point.html)

    **Yol Mesajı Yapısı**

    Bu yol mesajı, harita üzerindeki tüm yol noktalarının koordinatlarını içeren bir diziyi barındırır. Tüm noktalar, `poses` alanında `geometry_msgs/Point[]` türünde saklanır ve planlanan veya gerçekleştirilen rotaya ait konumsal bilgileri hassas bir şekilde temsil eder.

4. **Transform Topic**
   
   Topic: `tf_websocket`

   Msg_type: `geometry_msgs/TransformStamped`

   Msg_definition:

   [geometry_msgs/TransformStamped](https://docs.ros2.org/foxy/api/geometry_msgs/msg/TransformStamped.html)

   Robotun child_frame_id: base_link (Eger gelen mesajin child_frame_id == base_link robotta aittir)



5. **Keepout Polygons Topic**
   
   Topic: `server_keepout_polygons`

   Msg_type: `amr_websocket_interfaces/KeepoutPolygons`

   Msg_definition:

   ```json
    {
        "map_name": "string",
        "polygons": "amr_websocket_interfaces/PolygonPoints[]"
    }
   ```

   `amr_websocket_interfaces/PolygonPoints`:

   ```json
    {
        "points": "geometry_msgs/Point[]"
    }
   ```

   [geometry_msgs/Point](https://docs.ros.org/en/noetic/api/geometry_msgs/html/msg/Point.html)

   **Keepout Polygon Mesajı Açıklaması**

    KeepoutPolygons mesaj tipi, harita üzerindeki tüm kısıtlı veya geçişe kapalı bölgeleri temsil eder. `polygons` alanı, her biri `geometry_msgs/Point` türünde noktalar listesinden oluşan çokgenler kümesini içerir. Bu çokgenler, robotun navigasyon sırasında kaçınması gereken sınırları belirtir.

6. **Keepout Reference Topic**
   
   Topic: `keepout_points`

   Msg_type: `amr_websocket_interfaces/KeepoutReference`

   Msg_definition:

   ```json
    {
        "map_name": "string",
        "points": "geometry_msgs/Point[]"
    }
   ```

   Keepout Referans Noktaları Mesajı

    Bu mesaj, haritalama sırasında kullanılan keepout referans noktalarını temsil eder. Keepout bölgelerinin etkili ve doğru şekilde çizilebilmesi için referans noktalarına ihtiyaç vardır. Bu konu (topic), işaretlenen kısıtlı alanları yayınlayarak harita üzerindeki geçiş yasak bölgelerin net bir şekilde belirlenmesini sağlar.

7. **Stations Topic**
   
   Topic: `client_stations`

   Msg_type: `amr_websocket_interfaces/Stations`

   Msg_definition:

   ```json
    {
        "stations": "amr_websocket_interfaces/ClientStation[]"
    }
   ```

   `amr_websocket_interfaces/ClientStation`:

   ```json
    {
        "station_name": "string",
        "description": "string",
        "video": "string", //video_name.mp4 degil 'video_name'
        "position": "geometry_msgs/Point"
    }
   ```

   Bu mesajda istasyon adı, açıklaması, video adı (video_name.mp4 degil, video_name) ve haritadaki koordinatları yer almaktadır.

8. **Available maps Topic**
   
   Topic: `websocket_maps`

   Msg_type: `amr_websocket_interfaces/Maps`

   Msg_definition:

   ```json
    {
        "default_map": "string",
        "maps": "string[]"
    }
   ```

   Bu mesaj robotta bulunan tüm haritaları (`maps`) ve yüklü haritasını (`default_map`) gösterir. Varsayılan harita boşsa, kullanıcı arayüzü harita adını sormalıdır.

9. **Motor Driver Topic**
    
    Topic: `motor_driver_topic`

    Msg_type: `arduino_msgs/MotorDriver`

    Msg_definition:

    ```json
    {
        "motor_voltage": "float32",
        "left_motor_current": "float32",
        "right_motor_current": "float32",
        "driver_temp": "float32",
        "left_motor_temp": "int16",
        "right_motor_temp":"int16", 
        "encoder": "int32[]",
        "speed": "float32[]",
        "motor_status": "bool" 
    }
    ```

10. **Delivery Topic**
    
    Topic: `delivery_status`

    Msg_type: `amr_websocket_interfaces/DeliveryStatus`

    Msg_definition:

    ```json
    {
        "status": "string", 
        "completed_stations": "string[]",
        "pending_stations": "string[]"
    }
    ```
11. **Video Status Topic**
   
   Topic: `video_status`

   Msg_type: `orbit_command_msgs/VideoStatus`

   Msg_definition:

   ```json
    {
        "video_names": "string[]",
        "video_frames": "string[]"
    }
   ```

   Video Status Mesajı

    Video_names (string[]): Robotun içinde kayıtlı olan videoların isimlerini içeren bir dizi (liste). Her bir eleman, bir videonun adını temsil eder.

    Video_frames (string[]): Her bir video için, o videoya ait bir veya birden fazla görüntünün (frame) base64 formatında kodlanmış halini içeren bir dizi. Her eleman, bir görüntüyü base64 string olarak temsil eder.

12. **Short introductions Topic**
    
    Topic: `introductions`

    Msg_type: `amr_websocket_interfaces/Introductions`

    Msg_definition:

    ```json
     {
        "introductions": "amr_websocket_interfaces/ShortIntroduction[]"
     }
    ```

13. **Mode Status Topic**
   
   Topic: `robot_mode_name`

   Msg_type: `amr_websocket_interfaces/ModeControl`

   Msg_definition:

   ```json
    {
        "system_mode": "string",
        "nav2_mode": "string",
        "modes": "string",
        "system_variant": "string"
    }
   ```

   Mode Status Mesajı

    system_mode (string): "navigation" ya da "mat" eger mat modundaysa navigasyon sayfasi acilmaz. Bu mode control sayfasin da button ile gecisi saglanicaktir(Navigasyon sayfasi bittikten sonra suan sabit).

    nav2_mode (string): 'nav2' ya da 'slam' Robot eger modes AMCL modunda ise bu iki mode arasinda gecis yapilicak modes SLAM ise bu mesaj kullanilmayacak.

    modes (string): 'SLAM' ya da 'AMCL' robotun hangi localization sistemini kullandigi bilgisini aliyoruz. Kullanici tarafindan degistirilmiyor.

    system_variant (string): 'PRO' ya da 'LIGHT' robotun hangi versiyon bilgisini bize veriyor. Eger leight ise navigasyon moduyla alakali arayuzde ki hersey gozukucek ama kullanilamiyacak. Kullanmaya calisildiginda ise pro versiyonunu almaniz gerekli gibi bir uyari verebilir. Bu mesaj kullanici tarafindan degistirilemez.

14. **Short Introductions Status Topic**
    
    Topic: `shortintro_status`

    Msg_type: `amr_websocket_interfaces/ShortIntroStatus`

    Msg_definition:

    ```json
     {
        "status": "string"
     }

   Short Introductions Status Mesajı

    "status": Inactive, Active(Basladi), Moving(Hareket ediyor), Completed
    Failed, Canceled

15. **LaserScan Topic Topic**
    
    Topic: `scan_multi`

    Msg_type: `stds_msgs/LaserScan`

    Msg_definition:

    [stds_msgs/LaserScan](https://docs.ros.org/en/noetic/api/sensor_msgs/html/msg/LaserScan.html)

    Robotun engelleri gormesi icin kullanilir.
    Laserin child_frame_id: base_link (Eger gelen mesajin child_frame_id == base_link lasera aittir).
    
    4.Transform Topic

16. **Records Yayınlama**
   
   Topic: `records_data`

   Msg_type: `orbit_command_msgs/RecordsList`

   Msg_definition:

   ```json
    {
    "command": "Records[]"
    }
   ```
   Alt mesaj: Records

   ```json
    {
    "record_name": "string",
    "record_data": "string"
    }
   ```


   Açıklama:
    Sistemde kayıtlı olan tüm veriler RecordsList tipiyle bu topic üzerinden guncel olarak yayınlanır.

        command: Tüm kayıtları içeren liste.

        Her Records:

            record_name: Kayıt ismi.

            record_data: Kayıt içeriği.
17. **Motions Listesini Yayınlama**
   
   Topic: `motions_topic`

   Msg_type: `orbit_command_msgs/MotionArray`

   Msg_definition:

   ```json
    {
    "motions": "Motion[]"
    }
   ```
   Alt mesaj: Records

   ```json
    {
    "motions_text": "string"
    }
   ```

   Açıklama:
    Robotun desteklediği tüm hareketler bu topic üzerinden yayınlanır.

        motions: Hareket komutlarını içeren dizi.

        motions_text: Hareketin adı (örnek: "happy", "sad", "thinking").

18. **Motions Listesini Yayınlama**
   
   Topic: `all_movements`

   Msg_type: `orbit_command_msgs/MoveListAll`

   Msg_definition:

   ```json
    {
    "all_movements": "MoveListSave[]"
    }
   ```
   Alt mesaj: `MoveListSave`

   ```json
    {
    "movements_name": "string",
    "command": "Move[]"
    }
   ```
   Alt mesaj: `Move`

   ```json
    {
    "time": "float",
    "linear_x": "float",
    "angular_z": "float"
    }
   ```

   Açıklama:
    Sistemde kayıtlı tüm hareketleri yayınlar. Hareketlerin adı ve komut dizisi JSON'dan okunarak bu topic ile paylaşılır.

19. **All Task Yayınlama**
   
   Topic: `all_tasks`

   Msg_type: `orbit_command_msgs/TasksListAll`

   Msg_definition:

   ```json
    {
    "all_tasks": "TasksListSave[]"
    }
   ```
   Alt mesaj: `TasksListSave`

   ```json
    {
    "tasks_name": "string",
    "command": "Task[]"
    }
   ```

   Açıklama:
    JSON dosyasındaki tüm kayıtlı görevleri yayar.

20. **Motions Durumu Takibi**
   
   Topic: `arduino_topic`

   Msg_type: `arduino_msgs/Buttons`

   Msg_definition:

   ```json
    {
    "motions_status": "bool"
    }
   ```
   Açıklama:
    Arduino'dan gelen motion durumu bilgisi ile bir hareketin bitip bitmediği kontrol edilir.

21. **Ses Aygıtı Yayınlayıcı**
   
   Topic: `device`

   Msg_type: `orbit_command_msgs/Device`

   Msg_definition:

   ```json
    {
    "input_devices": "string[]",
    "output_devices": "string[]",
    "main_device": "string[2]"
    }
   ```
   Açıklama:
    Sistemdeki mikrofon (source) ve hoparlör (sink) aygıtlarını listeler.
    Ayrıca, o anki varsayılan giriş ve çıkış aygıtlarını main_device olarak gönderir.

        input_devices: Mevcut mikrofon aygıtlarının isimleri.

        output_devices: Mevcut hoparlör aygıtlarının isimleri.

        main_device: [mevcut varsayılan giriş, mevcut varsayılan çıkış] şeklinde iki elemanlı dizi.

22. **Volume Level Yayını**
   
   Topic: `current_volume`

   Msg_type: `std_msgs/Int32`

   Msg_definition:

   ```json
    {
    "data": "int32"
    }
   ```
   Açıklama:
    0.5 saniyede bir sistemdeki varsayılan çıkış ses seviyesi okunur ve değiştiyse bu topic üzerinden yayınlanır.

        data: Anlık çıkış ses seviyesi (% olarak).

        Değişiklik olmadıkça aynı değer tekrar yayınlanmaz. 0-150

22. **Compressed Image Publisher**
   
   Topic: `/camera/compressed/image`

   Msg_type: `sensor_msgs/CompressedImage`

   Msg_definition:

   ```json
    {
    "header": "std_msgs/Header",
    "format": "string",
    "data": "uint8[]"
    }
   ```
   Açıklama:
    Ham görüntü (Image) alındıktan sonra JPEG formatında %30 kaliteyle sıkıştırılır ve bu topic üzerinden yayınlanır.

        format: 'jpeg' olarak belirlenmiştir.

        data: JPEG sıkıştırılmış byte[] görüntü verisi.

        FPS: Görüntüler saniyede 10 kare (10 FPS) olarak yayınlanır.

23. **Set Volume**
   
   Topic: `current_task_loop`

   Msg_type: `orbit_command_msgs/TaskLoopStatus`

   Msg_definition:

   ```json
    {
    "total": "int32",      // Toplam döngü sayısı, -1 ise sonsuz döngü devam ediyor, 0 ise bitti.
    "remaining": "int32"   // Kalan döngü sayısı, -1 ise sonsuz döngü devam ediyor, 0 ise bitti.
    }
   ```

24. **Low Battery Warning Publisher**
   
   Topic: `low_battery`

   Msg_type: `std_msgs/Bool`

   Msg_definition:

   ```json
    {
    "data": "bool"
    }
   ```

   Açıklama:
    Eger data True gelirse ui a battery %10 altinda diyerek uyari penceresi acilmasi gerekli. False ise battery normal bir sey yapmaya gerek yok.

25. **Sensor Check Control Publisher**
   
   Topic: `system_errors`

   Msg_type: `system_control/Errors`

   Msg_definition:

   ```json
    {
    "camera_error": "int8",
    "front_lidar_error": "int8",
    "back_lidar_error": "int8",
    "imu_error": "int8",
    "motor_error": "int8",
    "arduino_error": "int8",
    "system_error": "int8"
    }
   ```

   Açıklama:
    `0:Hata Yok`, `1:Hata Var`

    Eger camera_error,front_lidar_error,back_lidar_error,imu_error,motor_error,arduino_error mesajlarindan en az birinden 1 mesaji gelirse Warning Ledi yanicak ve uyari penceresi gelicek. 
    
    Eger system_error 1 ise Error ledi yanicak ve error penceresi gelicek.

26. **Picture Base64 Publisher**
   
   Topic: `picture_base64`

   Msg_type: `std_msgs/String`

   Msg_definition:

   ```json
    {
    "data": "String"
    }
   ```
   Açıklama:
    Picture Task Servicenin tetiklendikten sonra publish edilir. Her mesaj geldiginde galeriye kaydedilir.

## Subscribers

1. **cmd_vel Topic**
   
   Topic: `cmd_vel`

   Msg_type: `geometry_msgs/Twist`

   [geometry_msgs/Twist](https://docs.ros.org/en/noetic/api/geometry_msgs/html/msg/Twist.html)

   Robot sürüşü için kullanılır.

2. **WebSocket Stations Topic**
   
   Topic: `websocket_stations`

   Msg_type: `amr_websocket_interfaces/AmrStations`

   Msg_definition:

   ```json
    {
        "map_name": "string",
        "station_name": "string",
        "description": "string",
        "video": "string", //video_name.mp4 degil 'video_name'
        "pose": "geometry_msgs/Pose"
    }
   ```

   Belirtilen koordinatlardaki bir istasyonu kaydetmek için bu topic'e yayınlayın.

3. **Station at current pose Topic**
   
   Topic: `station_at_current_pose`

   Msg_type: `amr_websocket_interfaces/AmrStations`

   `pose` alanı boş bırakılabilir.

4. **Delete Station Topic**
   
   Topic: `delete_station`

   Msg_type: `std_msgs/String`

   [std_msgs/String](https://docs.ros2.org/foxy/api/std_msgs/msg/String.html)

   Silinecek istasyon adı.

5. **Edit Station Topic**
   
   Topic: `edit_station`

   Msg_type: `amr_websocket_interfaces/AmrStations`

   İstasyon videosunu ve açıklamasını düzenlemek için kullanılır.

6. **Keepout Polygons Client Topic**
   
   Topic: `keepout_polygons`

   Msg_type: `amr_websocket_interfaces/KeepoutPolygons`

   istemciden gelen tüm tanımlanmış kısıtlı alan poligon noktaları.

7. **JoyStick Topic**
   
   Topic: `joy`

   Msg_type: [sensor_msgs/Joy](https://docs.ros2.org/foxy/api/sensor_msgs/msg/Joy.html)

   kısıtlı alan referansını işaretlemek için `buttons` dizisindeki 4. indeksi kullanın.

8. **Estimate on Station Topic**
   
   Topic: `amr_websocket/start_on_station`

   Msg_type: [std_msgs/String](https://docs.ros2.org/foxy/api/std_msgs/msg/String.html)

   `data`: istasyon ismi

9. **Task Stations Topic**
    
    Topic: `task_stations`

    Msg_type: `amr_websocket_interfaces/TaskStations`

    Msg_definition:

    ```json
     {
        "mode": "uint8",
        "stations": "string[]"
     }
    ```
    `uint8 NORMAL=0`: normal istasyon ziyareti.

    `uint8 INTRODUCTORY=1`: istasyonu ziyaret edin ve tanıtın.

10. **Save Short Introduction Topic**
    
    Topic: `save_introduction`

    Msg_type: `amr_websocket_interfaces/ShortIntroduction`

    Msg_definition:

    ```json
     {
        "introduction_name": "string",
        "description": "string",
        "video": "string", //video_name.mp4 degil 'video_name'
        "poses": "geometry_msgs/Pose[]"
     }
    ```

    [geometry_msgs/Pose](https://docs.ros2.org/foxy/api/geometry_msgs/msg/Pose.html)

11. **Start Short Introduction Topic**
    
    Topic: `start_introduction`

    Msg_type: [std_msgs/String](https://docs.ros2.org/foxy/api/std_msgs/msg/String.html)

    `data`: tanıtım adı.

12. **Delete Short Introduction Topic**
    
    Topic: `delete_introduction`

    Msg_type: [std_msgs/String](https://docs.ros2.org/foxy/api/std_msgs/msg/String.html)

13. **Contiune Button Topic**
    
    Topic: `arduino_topic`

    Msg_type: `arduino_msgs/Buttons`

    Msg_definition:

    ```json
     {
        "start_btn": "uint8",
     }
    ```
    `uint8 start_btn=1`: Devam etmek icin 1 gonderilir 

14. **Records Ekleme/Güncelleme**
   
   Topic: `records`

   Msg_type: `orbit_command_msgs/Records`

   Msg_definition:

   ```json
    {
    "record_name": "string",
    "record_data": "string"
    }
   ```

   Açıklama:
    Yeni bir kayıt eklemek veya var olan bir kaydı güncellemek için bu topic'e mesaj gönderilir.

        record_name: Kaydın ismi (örnek: "hello").

        record_data: Kayda ait içerik (örnek: bir json veya string ifade).

15. **Records Silme**
   
   Topic: `records_delete`

   Msg_type: `orbit_command_msgs/Records`

   Msg_definition:

   ```json
    {
    "record_name": "string",
    "record_data": "string"
    }
   ```

   Açıklama:
    Verilen isimdeki kayıt records.json dosyasından silinir. record_data alanı bu işlemde kullanılmaz.

        record_name: Silinecek kaydın ismi.

16. **Movements Komutu Yayınlama**
   
   Topic: `cmd_vel`

   Msg_type: `geometry_msgs/Twist`

   Msg_definition:[geometry_msgs/Twist](https://docs.ros.org/en/noetic/api/geometry_msgs/html/msg/Twist.html)

   Açıklama:
    Hareket komutları yürütülürken robotu doğrudan hareket ettirmek için cmd_vel üzerinden hız verisi yayınlanır.

17. **Movements Yürütme Komutu**
   
   Topic: `movements`

   Msg_type: `orbit_command_msgs/MoveList`

   Msg_definition:

   ```json
    {
    "command": "Move[]"
    }
   ```
   Alt_Msg: `Move`

   ```json
    {
    "time": "float",
    "linear_x": "float",
    "angular_z": "float"
    }
   ```

   Açıklama:
    Robotun hareket komutlarını çalıştırmak için kullanılan abonelik. Her Move, belirli bir süre (time) boyunca linear_x ve angular_z değerleri ile robotu hareket ettirir.

18. **Movements Kaydetme**
   
   Topic: `movements_datas`

   Msg_type: `orbit_command_msgs/MoveListSave`

   Msg_definition:

   ```json
    {
    "movements_name": "string",
    "command": "Move[]"
    }
   ```
   Alt_Msg: `Move`

   ```json
    {
    "time": "float",
    "linear_x": "float",
    "angular_z": "float"
    }
   ```

   Açıklama:
    Yeni bir hareket dizisini sistemde movements.json içerisine kaydeder veya günceller.

        movements_name: Kaydın ismi.

        command: Hareket dizisini tanımlayan Move listesi.

19. **Movements Silme**
   
   Topic: `movements_delete`

   Msg_type: `std_msgs/String`

   Msg_definition:

   ```json
    {
    "data": "string"
    }
   ```

   Açıklama:
    Verilen isimdeki hareket kaydını siler.

        data: Silinecek hareketin adı.

20. **Task Listesi Yurutme Komutu**
   
   Topic: `tasks_topic`

   Msg_type: `orbit_command_msgs/TasksList`

   Msg_definition:

   ```json
    {
    "command": "Task[]"
    }
   ```
   Alt_Msg: `Task`

   ```json
    {
    "message_type": "int32",
    "face": "string",
    "record": "string",
    "motion": "string",
    "command": "Twist[]"
    }
   ```

   Açıklama:
    Robotun ardışık görevler dizisini yürütmesi için bu topic'e görev listesi gönderilir.

    Görevler sırasıyla çalıştırılır. Her görev tipi (message_type) farklı işlevi tetikler:

    1: Yüz animasyonu (Face)

    2 ve 4: Kayıt işlemi (Records)

    3: Hareket (Motion)

    5: Robot hareket komutları

    6: Video oynatma

21. **Task Kaydetme**
   
   Topic: `tasks_datas`

   Msg_type: `orbit_command_msgs/TasksListSave`

   Msg_definition:

   ```json
    {
    "tasks_name": "string",
    "command": "Task[]"
    }
   ```

   Açıklama:
    Yeni bir görev dizisini sistemde tasks.json içine kaydeder veya var olanı günceller.
22. **Task Kaydetme**
   
   Topic: `tasks_delete`

   Msg_type: `std_msgs/String`

   Msg_definition:

   ```json
    {
    "data": "string"
    }
   ```

   Açıklama:
    Verilen isimdeki görevi kalıcı olarak siler.

23. **Motion Komutu Tetikleme**
   
   Topic: `motions`

   Msg_type: `std_msgs/String`

   Msg_definition:

   ```json
    {
    "data": "string"
    }
   ```

   Açıklama:
    Kafa hareket (motion) komutlarını tetiklemek için bu topic kullanılır.

24. **Set Volume**
   
   Topic: `set_volume`

   Msg_type: `std_msgs/Int32`

   Msg_definition:

   ```json
    {
    "data": "int32"
    }
   ```

   Açıklama:
    Bu topic'e gönderilen Int32 değeri (örneğin 85) sistemdeki varsayılan çıkış ses seviyesini (@DEFAULT_SINK@) ayarlamak için kullanılır.
    Geçerli aralık: 0 ile 150 arası (% cinsinden).

        data: Yeni ayarlanacak ses seviyesi yüzdesi (% olarak).

25. **Set Task Loop**
   
   Topic: `task_loop`

   Msg_type: `std_msgs/Int32`

   Msg_definition:

   ```json
    {
    "data": "int32"
    }
   ```

   Açıklama:
     0 : Döngü durdur,
    -1 : Sonsuz döngü,
    >0 : Belirtilen sayıda döngü

## Services
1. **Load Map**
   
   Service_name: `/amr_websocket/load_map`
   
   Service_type: `amr_websocket_interfaces/MapName`

   Service_definition:

   ```json
   # Request
    {
        "filename": "string"
    }
   ```
   ```json
   # Response
   0: Success
   1: Fail
    {
        "result": "uint8" 
    }
   ```

2. **Delete Map**
   
   Service_name: `/amr_websocket/delete_map`
   
   Service_type: `amr_websocket_interfaces/MapName`

   Service_definition:

   ```json
   # Request
    {
        "filename": "string"
    }
   ```
   ```json
   # Response
   0: Success
   1: Fail
    {
        "result": "uint8" 
    }
   ```

3. **Save Map**
   
   Service_name: `/amr_websocket/save_map`
   
   Service_type: `amr_websocket_interfaces/MapName`

   Service_definition:

   ```json
   # Request
    {
        "filename": "string"
    }
   ```
   ```json
   # Response
   0: Success
   1: Fail
    {
        "result": "uint8" 
    }
   ```

   Aciklama

    `Default_map bos ise ilk defa default_map ismi girilirken mutlaka service cagirilmali responsenin 0 geldiginden emin olunmali eger 1 ise tekrar denenmeli.`

4. **Change Mode**
   
   Service_name: `/set_mode_status`
   
   Service_type: `amr_websocket_interfaces/ModeStatus`

   Service_definition:

   ```json
   # Request
    {
        "mode": "string",
        "navmode": "string"
    }
   ```
   ```json
   # Response
   response: "mode_name", navmoderes: "navmode_name": Success
   response: "failed", navmoderes: "failed": Failed
    {
        "response": "string",
        "navmoderes": "string"
    }
    ```

    Aciklama

    Neye göre gönderilecek

        Mode Status Topic Publisher'dan gelen verilere bakılarak;

            "mode": system_mode

            "navmode":
            Eğer modes SLAM ise kullanılmayacak, restart mode servisi kullanılacak.
            Eğer modes AMCL ise:
                -nav_mode slam ise nav2 gönderilecek.
                -nav_mode nav2 ise slam gönderilecek.

5. **Restart Mode**
   
   Service_name: `/restart_system`
   
   Service_type: `std_srvs/Trigger`

   Service_definition:

   [std_srvs/Trigger](https://docs.ros.org/en/noetic/api/std_srvs/html/srv/Trigger.html)

        Kullanilagi Zamanlar

        Mode Status Topic Publisherdan gelen verilere bakilarak;
        Eger modes SLAM ise tekrar harita olusturmak icin kullanilacak.
        
        Eger modes AMCL ve nav_mode slam ise tekrardan harita olusturmak icin kullanilacak. Diger durumlarda kullanilmayacak.

6. **Cancel Service**
   
   Service_name: ` /navigate_to_pose/_action/cancel_goal`
   
   Service_type: `action_msgs/CancelGoal`

   Service_definition:

   [action_msgs/CancelGoal](https://docs.ros2.org/foxy/api/action_msgs/srv/CancelGoal.html)

7. **Face Service**
   
   Service_name: `/change_face` `Burda response feelings bitmeden hemen gelir`

   Service_name: `/face_task` `Burda response feelings bittikten sonra gelir.`
   
   Service_type: `orbit_command_msgs/Face`

   Service_definition:

   ```json
   # Request
    {
    "face": "uint8"
    }
   ```
   ```json
   # Response
   response: true : Success
   response: false : Failed
    {
    "response": "bool"
    }
    ```

    Açıklama:
        Robotun yüz ifadesini değiştirmek veya ilgili görseli göstermek için kullanılır.
        FACE_IDS = {
            0: "blinking.mp4",
            1: "breathe.mp4",
            2: "compassion.mp4",
            3: "curious.mp4",
            4: "error.mp4",
            5: "heart_eyes.mp4",
            6: "hello.mp4",
            7: "loading.mp4",
            8: "playful.mp4",
            9: "shy.mp4",
            10: "star_eyes.mp4",
            11: "surprised.mp4",
            12: "thank_you.mp4"
        }

8. **Records Service**
   
   Service_name: `/record` `Burda response record bitmeden hemen gelir`

   Service_name: `/record_task` `Burda response record bittikten sonra gelir.`
   
   Service_type: `orbit_command_msgs/Records`

   Service_definition:

   ```json
   # Request
    {
    "record": "string"
    }
   ```
   ```json
   # Response
   response: true : Success
   response: false : Failed
    {
    "response": "bool"
    }
    ```

    Açıklama:
        Texti sese cevirmek icin kullanilir, record requestine seslendirmek istedigimiz texti gondeririz.
    
9. **Set Audio Device**
   
   Service_name: `/set_input_device`
   
   Service_type: `orbit_command_msgs/SetDevice`

   Service_definition:

   ```json
   # Request
    {
    "input_name": "string",
    "output_name": "string"
    }
   ```
   ```json
   # Response
   response: true : Success
   response: false : Failed
    {
    "response": "bool"
    }
    ```

    Açıklama:
        Bu servis, varsayılan giriş (input) ve çıkış (output) ses aygıtlarını pactl komutları aracılığıyla değiştirir.
        Servis, belirtilen cihaz sistemde zaten aktifse değiştirmez, farklıysa aygıtı günceller.
        
        Servisi kullanirken input_name degisicekse output_name default_output olmali ki hata vermesin ayni sekilde digeri icin gecerli.
        
        Default_output ve default_input isimlerini Ses Aygıtı Yayınlayıcı alabilirsin.

            input_name: Değiştirilecek mikrofon aygıt adı (pactl list short sources çıktısındaki değer).

            output_name: Değiştirilecek hoparlör aygıt adı (pactl list short sinks çıktısındaki değer).

            response: İşlem başarılıysa true, hata varsa false.


10. **MP3 Service**
   
   Service_name: `/mpname`
   
   Service_type: `orbit_command_msgs/srv/Mpname`

   Service_definition:

   ```json
   # Request
    {
    "mpname": "string"
    }
   ```
   ```json
   # Response
   response: true : Success
   response: false : Failed
    {
    "response": "bool"
    }
    ```

    Açıklama:
        mpname: "battery"Motor Driver Publisherdan gelen batarya seviyesi 21.0 25.2 yuzde 10 un altina dustugunde gonderilir, 'camera_voice' ise fotograf cekiminde kullanilir.

11. **Remove Video**
   
   Service_name: `/remove_videos`
   
   Service_type: `orbit_command_msgs/srv/VideoRemove`

   Service_definition:

   ```json
   # Request
    {
    "videoname": "string"
    }
   ```
   ```json
   # Response
   response: true : Success
   response: false : Failed
    {
    "response": "bool"
    }
    ```

    Açıklama:
        video_name: "video.mp4"

12. **Start Video**
   
   Service_name: `/video_play`
   
   Service_type: `orbit_command_msgs/Video`

   Service_definition:

   ```json
   # Request
    {
    "videoname": "string"
    }
   ```
   ```json
   # Response
   response: true : Success
   response: false : Failed
    {
    "response": "bool"
    }
    ```

    Açıklama:
        video_name: "video.mp4" response request gonderdikten hemen sonra geliyor.
13. **Task Start Video**
   
   Service_name: `/video_play_task`
   
   Service_type: `orbit_command_msgs/Video`

   Service_definition:

   ```json
   # Request
    {
    "videoname": "string"
    }
   ```
   ```json
   # Response
   response: true : Success
   response: false : Failed
    {
    "response": "bool"
    }
    ```

    Açıklama:
        video_name: "video.mp4" response video bittikten sonra geliyor. Task icin bu kullaniliyor.


14. **Task Cancel Mode**
   
   Service_name: `/canceled_task`
   
   Service_type: `std_srvs/Trigger`

   Service_definition:

   [std_srvs/Trigger](https://docs.ros.org/en/noetic/api/std_srvs/html/srv/Trigger.html)

        Kullanilagi Zamanlar

        ShortIntronun cancel yapmak icin kullanilir. Starta basildiginda basta baslar.

15. **Picture Service**
   
   Service_name: `/picture_take`
   
   Service_type: `std_srvs/Trigger`

   Service_definition:

   [std_srvs/Trigger](https://docs.ros.org/en/noetic/api/std_srvs/html/srv/Trigger.html)

        Kullanilagi Zamanlar

        Fotograf cekmek icin kullanilir.Ve Picture publisherdan base64 verisi alinir ve kaydedilir.    

