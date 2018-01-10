# Introduction
<img src="https://github.com/rt106/rt106.github.io/raw/master/rt106.png" width="150"  title="Rt 106 Logo" align="left"> Rt 106 is a scalable, containerized analytics and application framework for creating interactive web applications around long running (batch style) analytics. The Rt 106 SDK and base Docker images allow analytic developers to author analytics in any language (C++, Python, Java) while the Rt 106 Angular and Node.js services allow application developers to author custom end-user web applications. Rt 106 uses service discovery and self description to dynamically add (and remove) analytics and their UI's to applications, analytic specific job queues for scaling analytic executions, and an abstract REST interface to a bulk datastore to deploy in a variety of environments (desktop, cloud). Rt 106 provides a data model that separates source/primary data from result/derived data and analytic execution provenance so analytic results can be documented and reproduced.

# Documentation
* [Algorithm SDK](ALGORITHM_SDK.md) - How to build a container around an algorithm that interfaces with Rt 106
* [Custom Application SDK](CUSTOM_APPLICATION_SDK.md) - How to build a custom user experience in front of Rt 106
* [API Reference](REFERENCE.md) - Full API reference
