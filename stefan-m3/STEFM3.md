# Part 2 - Flask

## Architectural Style - Flask' Services and Micro Services

The [Flask][1] application is a micro web framework written in Python. Flask alone does not have many advanced features, such as database abstractions, network protocols, or form validation, which contrasts other web frameworks like Node.js. Flask leaves these Architectural Decisions in the developer's hands, only supporting them with plugins. With this said, the developers at Flask constrain this application to fit the description of being 'micro' - extensive and straightforward. The Services and Microservices CC diagram below shows how truly dense this application is at runtime. In the scenario below, an example web app in the Examples directory uses the Flask Framework.

## Connector & Component (CC) Diagram

![CC View of Flask](./cc_veiw_flask.png)
**Figure X. CC Diagram of the Flask Runtime**

## About the Architectural Style

Flask's Architectural Style is compact, dense, and fixed. The CC runtime diagram reflects this style, as there are relatively few components and each component is highly connected. Also, these connections are not distributed evenly throughout the system, as some "God Class" like modules have more dependencies than others. However, this style achieves the Framework's goal by serving the sole purpose of connecting macroscopic components and plugins, while being lightweight and functional. Developers who use Flask do not intend to modify the Framework itself and expect that it is functional. Since this Framework is small and straightforward, the Flask Developer can more easily guarantee functionality, as there is less code and modules that bugs or defects could manifest. Although software architects expect modularity, Flask makes up for this with extensive documentation, instructions, and ease of use. There is a clear trade-off of modifiablity by focusing on usability. Hence, changes to this system are highly constrained with this design. As mentioned before, the point of a framework is to offer consistency and should not change on a user-to-user basis. This constraint achieves Flask's goal of usability, as it allows for a high flexibility on the macroscopic level and eases the developer's concern for the functionality on the microscopic level.

[1]: https://flask.palletsprojects.com/en/2.0.x/ "Flask Documentation"
