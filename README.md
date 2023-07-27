import folium

# 经纬度坐标序列
coordinates = [
    (40.7128, -74.0060),  # 纽约市
    (34.0522, -118.2437),  # 洛杉矶
    # 可以添加更多的经纬度坐标
]

# 创建一个地图对象
map_center = coordinates[0]  # 使用第一个坐标作为地图中心
map_obj = folium.Map(location=map_center, zoom_start=5)

# 在地图上添加轨迹
folium.PolyLine(locations=coordinates, color='blue').add_to(map_obj)

# 显示地图
map_obj.save('map_with_trajectory.html')
