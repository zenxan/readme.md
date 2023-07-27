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
