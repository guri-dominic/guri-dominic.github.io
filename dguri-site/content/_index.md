+++
title = "Home"
+++

My name is Dominic Guri. I earned my Ph.D. in Robotics from Carnegie Mellon University’s Robotics Institute, where my research focused on designing, evaluating, and deploying mobile manipulation systems for cluttered agricultural environments.

# Publications List


<style>
.publication {
  display: grid;
  grid-template-columns: 200px minmax(0, 1fr);
  column-gap: 36px;
  align-items: center;
  margin: 30px 0;
}

.publication-thumbnail {
  display: block;
  width: 200px;
  height: auto;
  margin: 0;
}

.publication-image-link {
  display: block;
}

.publication-image-link img {
  display: block;
  margin: 0;
}

.publication-details {
  min-width: 0;
  line-height: 1.25;
}

.publication-title {
  display: inline-block;
  margin-bottom: 2px;
  color: #1685e5;
  font-size: 1.05rem;
  font-weight: 700;
  line-height: 1.2;
  text-decoration: none;
}

.publication-title:hover {
  text-decoration: underline;
}

.publication-authors {
  font-size: 1rem;
}

.publication-venue {
  font-style: italic;
}

.publication-links {
  margin-top: 2px;
}

.publication-links a {
  color: #1685e5;
}

@media (max-width: 600px) {
  .publication {
    grid-template-columns: 1fr;
    row-gap: 16px;
    align-items: start;
  }

  .publication-thumbnail {
    width: 180px;
  }
}
</style>



<!-- {{ publication(
  title="A Systematic Robot Design Optimization Methodology with Application to Redundant Dual-Arm Manipulators",
  authors="<strong>Dominic Guri</strong> and George Kantor",
  venue="arXiv",
  year=2025,
  image="images/publications/dual_arm_system.png",
  image_alt="Diagram illustrating the dual-arm robot design methodology",
  paper_url="https://arxiv.org/abs/2507.21896",
  pdf_url="https://arxiv.org/pdf/2507.21896",
  code_url="",
  bibtex_url=""
) }} -->

{{ publication(
  title="A Systematic Robot Design Optimization Methodology with Application to Redundant Dual-Arm Manipulators",
  authors="<strong>Dominic Guri</strong> and George Kantor",
  venue="arXiv",
  year=2025,
  image="images/publications/DualArmv02.png",
  image_alt="Dual-arm robot design optimization diagram",
  paper_url="https://arxiv.org/abs/2507.21896",
  paper_label="arXiv",
  pdf_url="https://arxiv.org/pdf/2507.21896",
  code_url="",
  website_url="",
  video_url="",
  bibtex_url="",
  description=""
) }}

{{ publication(
  title="ODE Methods for Computing One-Dimensional Self-Motion Manifolds",
  authors="<strong>Dominic Guri</strong> and George Kantor",
  venue="arXiv",
  year=2025,
  image="images/publications/SMMv01.png",
  image_alt="Robot self-motion manifold visualization",
  paper_url="https://arxiv.org/abs/2507.21957",
  paper_label="arXiv",
  pdf_url="https://arxiv.org/pdf/2507.21957",
  code_url="",
  website_url="",
  video_url="https://youtu.be/G7qXyn-qM0s",
  bibtex_url="",
  description="Computing self-motion manifolds of redundancy 1 for 7DOF and 6DOF manipulators."
) }}

{{ publication(
  title="Towards Autonomous Crop Monitoring: Inserting Sensors in Cluttered Environments",
  authors="Moonyoung Lee, Aaron Berger, <strong>Dominic Guri</strong>, Kevin Zhang, Lisa Coffey, George Kantor, and Oliver Kroemer",
  venue="arXiv",
  year=2024,
  image="images/publications/cornbot.png",
  image_alt="Agricultural robot inserting sensors into cornstalks",
  paper_url="https://arxiv.org/abs/2311.03697",
  paper_label="arXiv",
  pdf_url="https://arxiv.org/pdf/2311.03697",
  code_url="",
  website_url="",
  video_url="",
  bibtex_url="",
  description=""
) }}

{{ publication(
  title="Hefty: A Modular Reconfigurable Robot for Advancing Robot Manipulation in Agriculture",
  authors="<strong>Dominic Guri</strong>, Moonyoung Lee, Oliver Kroemer, and George Kantor",
  venue="arXiv",
  year=2024,
  image="images/publications/hefty.png",
  image_alt="Hefty modular agricultural mobile manipulator",
  paper_url="https://arxiv.org/abs/2402.18710",
  paper_label="arXiv",
  pdf_url="https://arxiv.org/pdf/2402.18710",
  code_url="",
  website_url="https://kantor-lab.github.io/cornbot/",
  video_url="",
  bibtex_url="",
  description="The design process and systems integration for Hefty, a modular and reconfigurable utility robot for mobile manipulation in agriculture."
) }}

{{ publication(
  title="Runahead A*: Speculative Parallelism for A* with Slow Expansions",
  authors="Mohammad Bakhshalipour, Mohamad Qadri, <strong>Dominic Guri</strong>, Seyed Borna Ehsani, Maxim Likhachev, and Phillip B. Gibbons",
  venue="AAAI",
  year=2023,
  image="images/publications/run-ahead-astar.png",
  image_alt="Runahead A-star path-planning diagram",
  paper_url="https://ojs.aaai.org/index.php/ICAPS/article/download/27176/26949",
  paper_label="PDF",
  pdf_url="",
  code_url="https://github.com/cmu-roboarch/runahead-astar/",
  website_url="",
  video_url="",
  bibtex_url="",
  description=""
) }}

{{ publication(
  title="Racod: Algorithm/Hardware Co-design for Mobile Robot Path Planning",
  authors="Mohammad Bakhshalipour, Seyed Borna Ehsani, Mohamad Qadri, <strong>Dominic Guri</strong>, Maxim Likhachev, and Phillip B. Gibbons",
  venue="ACM",
  year=2022,
  image="images/publications/racod.png",
  image_alt="RACOD mobile robot path-planning diagram",
  paper_url="https://dl.acm.org/doi/pdf/10.1145/3470496.3527383",
  paper_label="PDF",
  pdf_url="",
  code_url="",
  website_url="",
  video_url="",
  bibtex_url="",
  description=""
) }}

{{ publication(
  title="Speculative Path Planning",
  authors="Mohammad Bakhshalipour, Mohamad Qadri, and <strong>Dominic Guri</strong>",
  venue="arXiv",
  year=2021,
  image="images/publications/speculative.png",
  image_alt="Comparison of conventional and speculative path planning",
  paper_url="https://arxiv.org/abs/2102.06261",
  paper_label="arXiv",
  pdf_url="https://arxiv.org/pdf/2102.06261",
  code_url="https://github.com/bakhshalipour/speculative-path-planning",
  website_url="",
  video_url="https://youtu.be/zf6-Sv3IwXg",
  bibtex_url="",
  description=""
) }}
