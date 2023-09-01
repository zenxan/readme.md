import numpy as np
import matplotlib.pyplot as plt

def bezier_curve(p0, p1, p2, t):
    return (1 - t)**2 * p0 + 2 * (1 - t) * t * p1 + t**2 * p2

# 定义直线a和b的起始和结束点
start_a = np.array([0, 0])
end_a = np.array([3, 0])
start_b = np.array([5, 2])
end_b = np.array([5, 5])

# 计算曲线c的控制点，使其和a、b的切线平滑连接
# control_point_1 = start_a + (end_a - start_a) * 0.33
# control_point_2 = start_b + (end_b - start_b) * 0.33
control_point_1 = np.array([5, 0])

# 生成贝塞尔曲线上的点
num_points = 100
t_values = np.linspace(0, 1, num_points)
curve_points = np.array([bezier_curve(end_a, control_point_1, start_b, t) for t in t_values])

# 绘制直线a、b和曲线c
plt.plot([start_a[0], end_a[0]], [start_a[1], end_a[1]], label='Line a')
plt.plot([start_b[0], end_b[0]], [start_b[1], end_b[1]], label='Line b')
plt.plot(curve_points[:, 0], curve_points[:, 1], label='Curve c')
plt.scatter([start_a[0], end_a[0], start_b[0], end_b[0]], [start_a[1], end_a[1], start_b[1], end_b[1]], color='red', label='Connection Points')
plt.xlabel('X')
plt.ylabel('Y')
plt.title('Smooth Connection between Lines')
plt.legend()
plt.grid()
plt.show()
