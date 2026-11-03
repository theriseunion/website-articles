[简体中文](./README.md) · [All articles](../../README.en.md) · [Read on the website](https://www.theriseunion.com/en/blog/riseunion-byd-autonomous-driving.html)

# RiseUnion Powers BYD's Autonomous Driving Evolution with High-Performance AI Compute

RiseUnion · March 20, 2026 at 08:00 (UTC+08:00)

RiseUnion built a high-performance, high-reliability autonomous driving model training compute foundation for BYD using Rise CAMP, solving GPU cluster utilization and storage cost challenges through coordinated scheduling and tiered data management.

As a global leader in new energy vehicles, **BYD** has ranked at the top of the industry in production and sales for multiple consecutive years, driven by its strong vertical integration and R&D capabilities. As autonomous driving enters the large model era, BYD's requirements for compute performance and utilization efficiency set an exceptionally high industry bar.


## The Compute Trap in Autonomous Driving Training

As autonomous driving tasks move into end-to-end training, "**deadlock**" risks in distributed tasks have become the primary source of compute waste. In traditional bare-metal setups, if some nodes aren't ready, the entire job stalls, leaving expensive AI compute clusters sitting idle.

Additionally, the massive volume of road-collected data demands extremely high storage performance. Using high-performance all-flash storage across the board makes overall infrastructure TCO unsustainable.


## Coordinated Scheduling: Maximizing Every Unit of Compute

To address these challenges, RiseUnion deployed [Rise CAMP](https://www.theriseunion.com/en/product/camp.html) to manage the project's high-performance AI compute nodes, ensuring efficient distributed training job scheduling through an intelligent framework.

The platform uses [topology-aware scheduling](https://www.theriseunion.com/en/blog/brain-02-topology-aware.html) to automatically detect the physical interconnect topology between hardware, matching compute tasks to optimal communication paths. It also features automatic job timeout and retry, ensuring long-cycle autonomous driving model training recovers quickly from transient failures, significantly reducing compute waste.

> Learn more about scheduling strategies: [Priority-Aware](https://www.theriseunion.com/en/blog/brain-01-priority-aware.html) · [Load-Aware](https://www.theriseunion.com/en/blog/brain-03-load-aware.html) · [Resource-Aware](https://www.theriseunion.com/en/blog/brain-04-resource-aware.html)


## Hot-Cold Tiering: Solving the Storage Cost Problem

To balance performance and cost, Rise CAMP implements a **hot-cold data tiering** mechanism. High-concurrency read/write hot data runs on all-flash arrays, while massive warm/cold data automatically flows to lower-cost hybrid-flash arrays. This strategy meets the I/O demands of autonomous driving model training while keeping overall storage investment under control.


## Business Value

By building this stable AI compute foundation, the customer significantly improved effective cluster uptime and reduced engineering debugging costs through visual monitoring tools. These technical capabilities shortened the end-to-end autonomous driving model delivery cycle, giving the customer a competitive edge in the fierce intelligent driving market.

---

[Read this article on the RiseUnion website](https://www.theriseunion.com/en/blog/riseunion-byd-autonomous-driving.html) · [Explore more articles](https://www.theriseunion.com/en/blog.html)
