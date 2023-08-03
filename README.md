<p align="center">
    <a href="https://www.agilelab.it/witboost">
        <img src="docs/img/witboost_logo.svg" alt="witboost" width=600 >
    </a>
</p>

Designed by [Agile Lab](https://www.agilelab.it/), witboost is a versatile platform that addresses a wide range of sophisticated data engineering challenges. It enables businesses to discover, enhance, and productize their data, fostering the creation of automated data platforms that adhere to the highest standards of data governance. Want to know more about witboost? Check it out [here](https://www.agilelab.it/witboost) or [contact us!](https://www.agilelab.it/contacts)

This repository is part of our [Starter Kit](https://github.com/agile-lab-dev/witboost-starter-kit) meant to showcase witboost's integration capabilities and provide a "batteries-included" product.

# Snowflake Output Port Template

- [Overview](#overview)
- [Usage](#usage)

## Overview

Use this template to automatically create an Output Port based on a Snowflake instance.

Refer to the [witboost Starter Kit repository](https://github.com/agile-lab-dev/witboost-starter-kit) for information on the Specific Provisioner that can be used to deploy components created with this Template.

### What's a Template?

A Template is a tool that helps create components inside a Data Mesh. Templates help establish a standard across the organization. This standard leads to easier understanding, management and maintenance of components. Templates provide a predefined structure so that developers don't have to start from scratch each time, which leads to faster development and allows them to focus on other aspects, such as testing and business logic.

For more information, please refer to the [official documentation](https://docs.witboost.agilelab.it/docs/p1_user/p6_advanced/p6_1_templates/#getting-started).

### What's an Output Port?

An Output Port refers to the interface that a Data Product uses to provide data to other components or systems within the organization. The methods of data sharing can range from APIs to file exports and database links.

### Snowflake

Snowflake is a cloud based data warehousing platform that simplifies the management and analysis of massive amounts of data. Its architecture separates computation and storage resources, allowing them to scale independently according to demand. The key features include:

- Storage and Compute Separation: Unlike many traditional databases, Snowflake separates compute and storage resources, allowing each to scale independently. This ensures you pay only for the resources you use.
- Multi-Cloud Capability: Snowflake is a platform-agnostic service that can be run on major cloud providers such as AWS, GCP and Microsoft Azure.
- Zero Management: Snowflake is offered as a fully-managed service, which means you don't have to worry about infrastructure, indexes, partitions, or tuning to maintain peak performance.
- Security: It provides robust security features such as end-to-end encryption, multi-factor authentication, and role-based access control to ensure data privacy and protection.
- Concurrency and Performance: Snowflake can handle large numbers of simultaneous users and queries without degrading performance.
- Time Travel: Snowflake allows you to access historical data at any point within a specified period, which can be up to 90 days. This can be used for data recovery or analytical purposes.
- Cloning: the clone capability allows us to quickly duplicate anything, including databases, schemas, tables, and other Snowflake objects, in almost real time.

Learn more about it on the [official website](https://www.snowflake.com/en/).

## Usage

### Prerequisites

A Data Product should already exist in order to attach the new components to it. The component **Snowflake Storage** should exist in the Data Product.

### Component basic information

This section includes the basic information that any Component of Data Mesh Boost must have

- Name: Required name used for display purposes on your Data Product
- Fully Qualified Name: Workload fully qualified name, this is optional as will be generated by the system if not given by you
- Description: A short description to help others understand what this Workload is for.
- Domain: The Domain of the Data Product this Workload belongs to. Be sure to choose it correctly as is a fundamental part of the Workload and cannot be changed afterwards.
- Data Product: The Data Product this Workload belongs to, be sure to choose the right one.
- Identifier: Unique ID for this new entity inside the domain. Don't worry to fill this field, it will be automatically filled for you.
- Development Group: Development group of this Data Product. Don't worry to fill this field, it will be automatically filled for you.
- Depends On: If you want your workload to depend on other components from the Data Product, you can choose this option.

  > In order for the Snowflake Output Port to create view on top of the tables, we need to add Snowflake Storage component in the **dependsOn** field.

*Example:*

| Field name              | Example value                                                                                           |
|:------------------------|:--------------------------------------------------------------------------------------------------------|
| **Name**                | Snowflake Vaccinations Output Port                                                                      |
| **Description**         | Offers insight in the COVID-19 Vaccinations                                                             |
| **Domain**              | domain:healthcare                                                                                       |
| **Data Product**        | system:healthcare.vaccinationsdp.0                                                                      |
| ***Identifier***        | Will look something like this: *marketing.healthcare.0.snowflake-vaccinations-output-port*              |
| ***Development Group*** | Might look something like this: *group:datameshplatform* Depends on the Data Product development group  |
| ***dependsOn***         | urn:dmb:cmp:healthcare:vaccinationsdp:0:snowflake-vaccinations-storage                                  |

### Provide Output Port deployment information

This section will tell the Output Port where to get the data from inside Snowflake

- Database: (Optional) Enter the name of the database you want to sync data into. If not specified the Domain name will be set as default value.
- Schema: (Optional) Enter the name of the default schema. If not specified the value `<DP_name>_<DP_version>` will be set as default value. (The version will be rendered without the dots)
- View name: Enter the name of the view that will be created inside the Snowflake Schema
- Table name: Enter the name of the table used as source to create the view on top of it

*Example:*

| Field name     | Example value           |
|:---------------|:------------------------|
| **Database**   | HEALTHCARE              |
| **Schema**     | vaccinationsdp          |
| **Table name** | vaccinations_clean      |
| **View name**  | vaccinations_clean_view |

### Snowflake Output port deployment information

The deployment of this component will create a Snowflake View inside the specified Database and Schema. In order to cover our use-case, described in the previous sections, we are assuming that by default the view and the other Snowflake resources will be created inside the default Database Schema, following the convention explained earlier.

Then, we have several options: as explained in the previous sections we are able to create futher multiple tables either with the Storage component or the SQL Workload (with a dedicated query inside the sql file). As a result, we can use one of these tables (or multiple tables) already provided by one of these components (or multiple components) to create our view on top of it(/them). Of course, in that case we need to specify the proper dependencies by adding the involved components inside the component DependsOn section of this component and change the input data accordingly, otherwise the Data Product deployment will not work as expected.

By default, this component will create a View on top of the Airbyte source table that will be created inside Snowflake. The View structure will follow the source table structure obviously, but the fields included will be the ones specified inside the component descriptor, listed under the `dataContract > schema` section of the `catalog-info.yaml` file. **You must change the tableName specified as source and the schema to customize your view**.

### Custom View

The user, as already mentioned, has also the possibility to create a custom view based on different/multiple tables. In addition, you can leverage the features offered by the View to apply **masking and row access policies**. In that case, the features explained until now will not do the job, and you may need a custom view.

To create a custom view, you simply need to provide the query needed to create it by manually editing the `catalog-info.yaml` file. In particular, you need to add it in a new `customView` field inside the `specific` section of the `catalog-info.yaml` file, as showed below:

!! Be aware that we had followed certain rules for naming the specifics inside the catalog-info file (For example - Added suffixes for schemaName and appended it with majorVersion) regardless of the value being a default one (or) a custom one. During any point of time, if you have any doubts regarding these names, please refer to the catalog-info file of **Snowflake Storage** as this would be the starting point of the tutorial.

```yaml
  components:
    - id: domain:healthcare.vaccinationsdp.0.snowflake-output-port
      name: vaccinationsdp
      fullyQualifiedName: Vaccinations Data
      description: Vaccinations Data
      kind: outputport
      owners:
        - group:bigdata
      infrastructureTemplateId: urn:dmb:itm:cdp-aws-s3:1.0.0
      useCaseTemplateId: urn:dmb:utm:template-id-1:1.0.0
      dependsOn: [ ]
      platform: CDP_AWS
      technology: CDP_S3
      dataContract:
        schema:
          - name: date
            dataType: DATE
            constraint: NO CONSTRAINT
          - name: location_key
            dataType: TEXT
            constraint: NO CONSTRAINT
          - name: new_persons_vaccinated
            dataType: NUMBER
            constraint: NO CONSTRAINT
          - name: new_persons_fully_vaccinated
            dataType: NUMBER
            constraint: NO CONSTRAINT
          - name: new_vaccine_doses_administered
            dataType: NUMBER
            constraint: NO CONSTRAINT
          - name: cumulative_persons_vaccinated
            dataType: NUMBER
            constraint: NO CONSTRAINT
          - name: cumulative_persons_fully_vaccinated
            dataType: NUMBER
            constraint: NO CONSTRAINT
          - name: cumulative_vaccine_doses_administered
            dataType: NUMBER
            constraint: NO CONSTRAINT
      tags: [ ]
      specific:
        viewName: vaccinations_clean_view
        tableName: vaccinations_clean
        database: HEALTHCARE
        schema: VACCINATIONSDP_0
        customView: CREATE VIEW IF NOT EXISTS HEALTHCARE.VACCINATIONSDP_0.vaccinations_clean_view AS (SELECT * FROM HEALTHCARE.VACCINATIONSDP_0.vaccinations_clean);
```

**Please note that when providing a customView query, the source tableName and the dataContract schema specified will be not used to create the View, but those parts must be aligned and compiled because the platform will check their consistency through policies.**

## License

This project is available under the [Apache License, Version 2.0](https://opensource.org/licenses/Apache-2.0); see [LICENSE](LICENSE) for full details.

## About us

<p align="center">
    <a href="https://www.agilelab.it">
        <img src="docs/img/agilelab_logo.jpg" alt="Agile Lab" width=600>
    </a>
</p>

Agile Lab creates value for its Clients in data-intensive environments through customizable solutions to establish performance driven processes, sustainable architectures, and automated platforms driven by data governance best practices.

Since 2014 we have implemented 100+ successful Elite Data Engineering initiatives and used that experience to create Witboost: a technology agnostic, modular platform, that empowers modern enterprises to discover, elevate and productize their data both in traditional environments and on fully compliant Data mesh architectures.

[Contact us](https://www.agilelab.it/contacts) or follow us on:
- [LinkedIn](https://www.linkedin.com/company/agile-lab/)
- [Instagram](https://www.instagram.com/agilelab_official/)
- [YouTube](https://www.youtube.com/channel/UCTWdhr7_4JmZIpZFhMdLzAA)
- [Twitter](https://twitter.com/agile__lab)