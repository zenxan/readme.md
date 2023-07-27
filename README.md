def plot_trajectory(coordinates):
    # 创建一个基本地图对象
    map_center = [sum(p[0] for p in coordinates) / len(coordinates), 
                  sum(p[1] for p in coordinates) / len(coordinates)]
    m = folium.Map(location=map_center, zoom_start=14)

    # 添加轨迹到地图上
    folium.PolyLine(locations=coordinates, color='blue').add_to(m)

    return m

if __name__ == "__main__":
    # 示例经纬度坐标序列
    # 请替换以下坐标序列为您自己的经纬度坐标
    coordinates_sequence = [
        (latitude_1, longitude_1),
        (latitude_2, longitude_2),
        # 添加更多坐标点...
    ]

    # 绘制轨迹并保存地图
    map_object = plot_trajectory(coordinates_sequence)
    map_object.save("trajectory_map.html")


beijing_coordinates = [
    (39.9042, 116.4074),  # 北京市中心
    (39.9496, 116.4336),  # 东城区
    (39.9232, 116.3962),  # 西城区
    (39.9624, 116.2984),  # 海淀区
    (39.8950, 116.4977),  # 朝阳区
    (39.8837, 116.2654),  # 丰台区
    (39.9255, 116.5544),  # 石景山区
    (39.9900, 116.4117),  # 通州区
    (39.8983, 116.3863),  # 顺义区
    (40.0027, 116.4074),  # 昌平区
    (39.7392, 116.3370),  # 大兴区
    (39.8099, 116.2673),  # 房山区
    (39.7728, 116.3683)   # 密云区
]


import sys
from cyber_py import cyber

def bag_reader(file_path):
    # Initialize cyber node
    cyber.init()

    # Create a reader to read the bag file
    bag = cyber.Bag(file_path)

    # Get the topic list from the bag
    topic_list = bag.get_topics()

    # Loop through each topic and read the messages
    for topic_name in topic_list:
        topic = bag.get_topic(topic_name)
        for msg in topic:
            # Process the message data
            # In this example, we will just print the topic name and the message data
            print(f"Topic: {topic_name}, Message: {msg}")

    # Close the bag
    bag.close()

if __name__ == "__main__":
    if len(sys.argv) != 2:
        print("Usage: python bag_reader.py <bag_file_path>")
        sys.exit(1)

    bag_file_path = sys.argv[1]
    bag_reader(bag_file_path)

