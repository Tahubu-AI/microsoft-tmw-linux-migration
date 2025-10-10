# Microsoft TMW Migration Lab

Contoso Inc., a global manufacturing and retail company, is embarking on a strategic initiative to modernize its aging on-premises infrastructure and embrace the scalability, resilience, and innovation offered by Microsoft Azure. Over the years, Contoso has accumulated a diverse set of workloads, including legacy virtual machines, custom web applications, and mission-critical databases, spread across multiple physical servers and data centers.

To reduce operational overhead, improve availability, and accelerate innovation, Contoso’s IT leadership has prioritized a phased migration to Azure. This workshop simulates key stages of that journey, focusing on real-world migration scenarios that span infrastructure, applications, and data platforms.

Workshop participants will step into the role of Contoso’s cloud migration team, tasked with evaluating, preparing, and executing targeted migrations using Azure-native tools and services. Each exercise represents a distinct workload type and migration strategy:

1. [Lift-and-shift migration of a Linux VM](./Labs/Exercise-01/readme.md): Contoso’s legacy Ubuntu-based human resources personnel system is migrated to an Azure VM to preserve compatibility while gaining cloud-based manageability and resilience.

2. [SQL Server migration to Azure SQL Database](./Labs/Exercise-02/readme.md): The SQL Server database hosting the human resources personnel data is migrated to Azure SQL Database to modernize Contoso’s data estate, reduce licensing costs, and enable integration with other Azure services.

3. [PostgreSQL database migration to Azure Database for PostgreSQL - Flexible Server](/Labs/Exercise-03/readme.md): The `dvdrental` database, backing Contoso’s customer facing PHP-based application is moved to Azure Database for PostgreSQL - Flexible Server to reduce maintenance overhead and improve scalability, while preserving compatibility with existing applications.

4. [Application replication to Azure App Service](./Labs/Exercise-04/readme.md): A PHP-based customer DVD rental application, hosted on a Linux VM, is replicated to an Azure App Service Plan, enabling Contoso to explore platform-as-a-service (PaaS) benefits such as autoscaling, integrated monitoring, and simplified deployment.

Throughout the workshop, participants will encounter realistic challenges such as authentication boundaries, network configuration, and role-based access control, mirroring the complexities faced by enterprise migration teams. By the end of the workshop, learners will have hands-on experience with multiple Azure migration paths and a deeper understanding of how to plan and execute cloud modernization initiatives.
