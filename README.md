import folium

def plot_trajectory(coords):
    # Create a map centered on the first coordinate
    map_center = [coords[0][0], coords[0][1]]
    my_map = folium.Map(location=map_center, zoom_start=15)

    # Plot the trajectory as a polyline on the map
    folium.PolyLine(coords, color='blue').add_to(my_map)

    # Mark the starting and ending points with markers
    folium.Marker(location=coords[0], popup='Start', icon=folium.Icon(color='green')).add_to(my_map)
    folium.Marker(location=coords[-1], popup='End', icon=folium.Icon(color='red')).add_to(my_map)

    # Display the map
    return my_map

# Example coordinates (replace this with your own list of coordinates)
coordinates_sequence = [
    (latitude1, longitude1),
    (latitude2, longitude2),
    (latitude3, longitude3),
    # Add more coordinates as needed
]

# Call the function to plot the trajectory
map_with_trajectory = plot_trajectory(coordinates_sequence)

# Save the map as an HTML file
map_with_trajectory.save('trajectory_map.html')
