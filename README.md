// gnss pose constraint
  if (options_.gnss_translation_weight > 0.0 ||
      options_.gnss_rotation_weight > 0.0) {
    cv::Mat matA(3, 3, CV_64F, cv::Scalar::all(0));
    cv::Mat matB(3, 1, CV_64F, cv::Scalar::all(0));

    matA.at<double>(0, 0) = options_.gnss_translation_weight;
    matA.at<double>(1, 1) = options_.gnss_translation_weight;
    matA.at<double>(2, 2) = options_.gnss_rotation_weight;

    matB.at<double>(0, 0) =
        options_.gnss_translation_weight * (match_pose_[0] - gnss_pose_[0]);
    matB.at<double>(1, 0) =
        options_.gnss_translation_weight * (match_pose_[1] - gnss_pose_[1]);
    matB.at<double>(2, 0) =
        options_.gnss_rotation_weight * (match_pose_[2] - gnss_pose_[2]);

    cv::Mat matAt(3, 3, CV_64F, cv::Scalar::all(0));
    cv::transpose(matA, matAt);
    matAtA += matAt * matA;
    matAtB -= matAt * matB;
  }
