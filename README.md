import folium
from folium import plugins

def plot_trajectory_with_amap(coordinates):
    # 创建一个基本地图对象
    map_center = [sum(p[0] for p in coordinates) / len(coordinates), 
                  sum(p[1] for p in coordinates) / len(coordinates)]
    m = folium.Map(location=map_center, zoom_start=14, control_scale=True)

    # 使用高德地图插件
    plugins.TileLayer('http://webst01.is.autonavi.com/appmaptile?style=6&x={x}&y={y}&z={z}', 
                      attr='AutoNavi', name='AutoNavi').add_to(m)

    # 添加轨迹到地图上
    folium.PolyLine(locations=coordinates, color='blue').add_to(m)

    return m

if __name__ == "__main__":
    # 示例经纬度坐标序列
    coordinates_sequence = [
        (latitude_1, longitude_1),
        (latitude_2, longitude_2),
        # 添加更多坐标点...
    ]

    # 绘制轨迹并保存地图
    map_object = plot_trajectory_with_amap(coordinates_sequence)
    map_object.save("trajectory_map_amap.html")
