
double LineSegment2d::GetPerpendicularFoot(const Vec2d &point,
                                           Vec2d *const foot_point) const {
  CHECK_NOTNULL(foot_point);
  if (length_ <= kMathEpsilon) {
    *foot_point = start_;
    return point.DistanceTo(start_);
  }
  const double x0 = point.x() - start_.x();
  const double y0 = point.y() - start_.y();
  const double proj = x0 * unit_direction_.x() + y0 * unit_direction_.y();
  *foot_point = start_ + unit_direction_ * proj;
  return std::abs(x0 * unit_direction_.y() - y0 * unit_direction_.x());
}
