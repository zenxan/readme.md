void ICPMatcher2D::AddResidualBlock(
    const std::vector<Constraint2D>& constraints, double weight,
    cv::Mat& matAtA, cv::Mat& matAtB) {
  double srz = sin(match_pose_[2]);
  double crz = cos(match_pose_[2]);

  cv::Mat matA(constraints.size(), 3, CV_64F, cv::Scalar::all(0));
  cv::Mat matB(constraints.size(), 1, CV_64F, cv::Scalar::all(0));
  for (size_t i = 0; i < constraints.size(); ++i) {
    const auto& position = constraints[i].position;
    const auto& direction = constraints[i].direction;
    const double arz =
        (-srz * position.x() - crz * position.y()) * direction.x() +
        (crz * position.x() - srz * position.y()) * direction.y();

    matA.at<double>(i, 0) = weight * direction.x();
    matA.at<double>(i, 1) = weight * direction.y();
    matA.at<double>(i, 2) = weight * arz;
    matB.at<double>(i, 0) = weight * constraints[i].distance;
  }

  cv::Mat matAt(3, constraints.size(), CV_64F, cv::Scalar::all(0));
  cv::transpose(matA, matAt);
  matAtA += matAt * matA;
  matAtB += matAt * matB;
}
