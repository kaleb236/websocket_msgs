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
        "video": "string",
        "position": "geometry_msgs/Point"
    }
   ```

   Bu mesajda istasyon adı, açıklaması, video adı ve haritadaki koordinatları yer almaktadır.

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
        "video": "string",
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
        "video": "string",
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

3. **Save Map**
   
   Service_name: `/amr_websocket/save_map`
   
   Service_type: `amr_websocket_interfaces/MapName`