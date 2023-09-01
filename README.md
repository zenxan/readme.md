def calculate_intersection(x1, y1, x2, y2, x3, y3, x4, y4):
    # 计算线段1和线段2的直线方程系数
    A1, B1, C1 = y2 - y1, x1 - x2, x2 * y1 - x1 * y2
    A2, B2, C2 = y4 - y3, x3 - x4, x4 * y3 - x3 * y4
    
    # 计算系数行列式
    det = A1 * B2 - A2 * B1
    
    # 如果系数行列式为0，则线段平行或不相交
    if det == 0:
        return None
    
    # 计算交点坐标
    x = (B2 * C1 - B1 * C2) / det
    y = (A1 * C2 - A2 * C1) / det
    # print(x,y)
    # 检查交点是否在两条线段上
    return -x, -y

# 给定两条线段的端点坐标
[x1, y1] = start_a
[x2, y2] = end_a
[x3, y3] = start_b
[x4, y4] = end_b

# 计算交点
intersection = calculate_intersection(x1, y1, x2, y2, x3, y3, x4, y4)
if intersection:
    print(f"两条线段的交点坐标为：{intersection}")
else:
    print("两条线段平行或不相交，没有交点。")

control_point_1 = np.array(intersection)
