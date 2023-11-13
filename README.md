#include <iostream>
#include <Eigen/Dense>
#include <vector>
#include <unordered_map>
// #include <execution>
// #include <opencv2/opencv.hpp>

using namespace std;
using namespace Eigen;

template <int N>
struct hash_vec
{
    inline size_t operator()(const Eigen::Matrix<int, N, 1> &v) const;
};

template <>
inline size_t hash_vec<2>::operator()(const Eigen::Matrix<int, 2, 1> &v) const
{
    return size_t(((v[0] * 73856093) ^ (v[1] * 471943)) % 10000000);
}

int bfnn_point(std::vector<Eigen::Vector2f> cloud, const Eigen::Vector2f &point)
{
    return std::min_element(cloud.begin(), cloud.end(),
                            [&point](const Eigen::Vector2f &pt1, const Eigen::Vector2f &pt2) -> bool
                            {
                                return (pt1 - point).squaredNorm() <
                                       (pt2 - point).squaredNorm();
                            }) -
           cloud.begin();
}

class GridNN
{
public:
    using KeyType = Eigen::Matrix<int, 2, 1>;
    using PtType = Eigen::Matrix<float, 2, 1>;
    using Vec2i = Eigen::Vector2i;

    enum class NearbyType
    {
        CENTER,
        NEARBY4,
        NEARBY8,
    };

    explicit GridNN(float resolution = 0.1, NearbyType nearby_type = NearbyType::NEARBY4) : resolution_(resolution), nearby_type_(nearby_type)
    {
        inv_resolution_ = 1.0 / resolution_;
        GenerateNearbyGrids();
    }

    bool SetBoundaryPoints(std::vector<Eigen::Vector2f> points);

    bool GetClosestPoint(const Eigen::Vector2f &pt, Eigen::Vector2f &closest_pt, size_t &idx);

    bool GetClosestPointForCloud(std::vector<Eigen::Vector2f> &points_ref, std::vector<Eigen::Vector2f> &points_query,
                                 std::vector<std::pair<size_t, size_t>> &matches);
    // bool GetClosestPointForCloudMT(std::vector<Eigen::Vector2f> &points_ref, std::vector<Eigen::Vector2f> &points_query,
    //                                std::vector<std::pair<size_t, size_t>> &matches);

private:
    void GenerateNearbyGrids();

    KeyType Pos2Grid(const PtType &pt);

    float resolution_ = 0.1;
    float inv_resolution_ = 1.0;

    NearbyType nearby_type_ = NearbyType::NEARBY4;
    std::unordered_map<KeyType, std::vector<size_t>, hash_vec<2>> grids_;
    std::vector<Eigen::Vector2f> points_;

    std::vector<KeyType> nearby_grids_;
};

void GridNN::GenerateNearbyGrids()
{
    if (nearby_type_ == NearbyType::CENTER)
    {
        nearby_grids_.emplace_back(KeyType::Zero());
    }
    else if (nearby_type_ == NearbyType::NEARBY4)
    {
        nearby_grids_ = {Vec2i(0, 0), Vec2i(-1, 0), Vec2i(1, 0), Vec2i(0, 1), Vec2i(0, -1)};
    }
    else if (nearby_type_ == NearbyType::NEARBY8)
    {
        nearby_grids_ = {Vec2i(0, 0), Vec2i(-1, 0), Vec2i(1, 0), Vec2i(0, 1), Vec2i(0, -1),
                         Vec2i(-1, -1), Vec2i(-1, 1), Vec2i(1, -1), Vec2i(1, 1)};
    }
}

bool GridNN::SetBoundaryPoints(std::vector<Eigen::Vector2f> points)
{
    std::vector<size_t> index(points.size());
    std::for_each(index.begin(), index.end(), [idx = 0](size_t &i) mutable
                  { i = idx++; });

    std::for_each(index.begin(), index.end(), [&points, this](const size_t &idx)
                  { 
                    Eigen::Vector2f pt = points[idx];
                    KeyType key = Pos2Grid(pt); 
                    if(grids_.find(key) == grids_.end()){
                        grids_.insert({key, {idx}});
                    }else{
                        grids_[key].emplace_back(idx);
                    } });
    points_ = points;
    return true;
}

Eigen::Matrix<int, 2, 1> GridNN::Pos2Grid(const Eigen::Matrix<float, 2, 1> &pt)
{
    return (pt * inv_resolution_).template cast<int>();
}

bool GridNN::GetClosestPointForCloud(std::vector<Eigen::Vector2f> &points_ref, std::vector<Eigen::Vector2f> &points_query,
                                     std::vector<std::pair<size_t, size_t>> &matches)
{
    matches.clear();
    std::vector<size_t> index(points_query.size());
    std::for_each(index.begin(), index.end(), [idx = 0](size_t &i) mutable
                  { i = idx++; });
    std::for_each(index.begin(), index.end(), [this, &matches, points_query](const size_t &idx)
                  {
        Eigen::Vector2f cp;
        size_t cp_index;
        if(GetClosestPoint(points_query[idx], cp, cp_index)){
            matches.emplace_back(cp_index, idx);
        } });
    return true;
}

bool GridNN::GetClosestPoint(const Eigen::Vector2f &pt, Eigen::Vector2f &closest_pt, size_t &idx)
{
    std::vector<size_t> idx_to_check;
    auto key = Pos2Grid(pt);
    std::for_each(nearby_grids_.begin(), nearby_grids_.end(), [&key, &idx_to_check, this](const KeyType &delta)
                  {
        auto dkey = key+delta;
        auto iter = grids_.find(key);
        if(iter != grids_.end()){
            idx_to_check.insert(idx_to_check.begin(), iter->second.begin(), iter->second.end());
        } });

    if (idx_to_check.empty())
        return false;

    std::vector<Eigen::Vector2f> nearby_points;
    std::vector<size_t> nearby_idx;
    for (auto &idx : idx_to_check)
    {
        nearby_points.emplace_back(points_[idx]);
        nearby_idx.emplace_back(idx);
    }

    size_t closest_point_idx = bfnn_point(nearby_points, pt);
    idx = nearby_idx.at(closest_point_idx);
    closest_pt = points_[idx];
    return true;
}

int main()
{
    std::vector<Eigen::Vector2f> points1;
    for (int i = 0; i < 100; i++)
    {
        float x = (float)i / 10.0;
        float y = 10*x;
        Eigen::Vector2f point(x, y);
        points1.emplace_back(point);
        // std::cout << "x " << point[0] << " y " << point[1] << std::endl;
    }

    std::vector<Eigen::Vector2f> points2;
    for (int i = 20; i < 80; i++)
    {
        float x = (float)i / 10.0;
        float y = 10*x+0.5;
        Eigen::Vector2f point(x, y);
        points2.emplace_back(point);
        // std::cout << "x " << point[0] << " y " << point[1] << std::endl;
    }

    Eigen::Matrix2f cov = Eigen::Matrix2f::Identity();
    Eigen::Matrix2f H = Eigen::Matrix2f::Identity();

    // for (int i = 0; i < 100; i++)
    // {
    //     Eigen::Vector2f noise_vec;
    //     noise_vec << 0.1 * i / 2, 0.1 * i / 2;
    //     Eigen::Matrix2f V = noise_vec.asDiagonal();
    //     Eigen::Matrix2f K = cov * H.transpose() * (H * cov * H.transpose() + V).inverse();
    //     Eigen::Vector2f p1 = points1[i];
    //     Eigen::Vector2f p2 = points2[i];
    //     float dx = p2[0] - p1[0];
    //     float dy = p2[1] - p1[1];
    //     Eigen::Vector2f innov(dx, dy);
    //     Eigen::Vector2f dp = K * innov;
    //     Eigen::Vector2f p = p1 + dp;
    //     // std::cout << "x " << p[0] << " y " << p[1] << std::endl;
    // }

    GridNN gridnn(1, GridNN::NearbyType::NEARBY8);
    std::vector<std::pair<size_t, size_t>> matches;
    gridnn.SetBoundaryPoints(points1);
    gridnn.GetClosestPointForCloud(points1, points2, matches);

    for (int i = 0; i < matches.size(); i++)
    {
        std::cout << matches[i].first << " " << matches[i].second << std::endl;
    }

    return 0;
}
