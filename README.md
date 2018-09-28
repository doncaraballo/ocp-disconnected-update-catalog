# Update Application Catalog

An OpenShift disconnection installation presents many challenges, many of which include developing a method to make the required OpenShift software registries and container images available to an environment that does not have access the Internet.  And as with any software project, updates will need to be made a system from time to time.  One such update could include making a newer software version container image release available to those utilizing the web console application catalog.  

By default, OpenShift provides a number of base container images.  However, the latest updates for those respective images will need to be added afterwards.  In this example, we will import the latest Python image into OpenShift and make it available to users as part of the application catalog.

## Importing a Container Image
 
OpenShift ships with the following Python container images (2.7, 3.3, 3.4, and 3.5).  As such, a developer may have a project that requires Python 3.6.  Images are tracked with using the Image Stream definition which exists in the "openshift" project.  Assuming the use of the Red Hat container registry the following command can be run:
```
oc import-image python:3.6 --from=registry.access.redhat.com/rhscl/python-36-rhel7:latest --confirm
```
The import-image command allows one to import the lastest image information.  In our example, we created the Python 3.6 tag in the "python" image stream (imagestream:tag).  The 3.6 image is imported from the registry.access.redhat.com repository.  The "--confirm" option is required to require OpenShift to create the Image Stream definition.  Please note that the container registry can be a public registry (such as registry.access.redhat.com, docker.io) or an external private registry (which would be the used as part of a disconnected installation).

To view the updated image stream for Python, the following command can be run:
```
oc describe is python -n openshift
```
The 3.6 tag should now be visible in the Image Stream:
```
Name:			python
Namespace:		openshift
Created:		2 months ago
Labels:			<none>
Annotations:		openshift.io/display-name=Python
			openshift.io/image.dockerRepositoryCheck=2018-09-28T22:12:41Z
Docker Pull Spec:	docker-registry.default.svc:5000/openshift/python
Image Lookup:		local=false
Unique Images:		6
Tags:			6

3.6
  tagged from registry.access.redhat.com/rhscl/python-36-rhel7:latest

  * registry.access.redhat.com/rhscl/python-36-rhel7@sha256:a6c84ddf8e4181c45343481e6d4bb792051f4d17dd65b84970b16dd7d19f3a86
      2 minutes ago
```
The newly created Image Stream definition for Python 3.6 is now available for use within OpenShift.  However, only while utilizing the CLI.  The image is not yet available for use from the web console.

## Image Stream Annotation

The next step to adding the Python image to the application catalog in the web console is to add an annotation.  Annotations are used to attach metadata to objects.  Per Kubernetes documentation, an annotation can be small, large, structured or unstructured, and can include a multitude of characters not permitted by labels.

One way to add or modify an annotation is to update the Image Stream definition and replace it with an updated version.  The Image Stream definition can by viewed using the following command:
```
oc get is python -o yaml
```

The following displays the definition for the 3.5 and 3.6 tags:
```
    referencePolicy:
      type: Source
  - annotations:
      description: Build and run Python 3.5 applications on RHEL 7. For more information about using this builder image, including OpenShift considerations, see https://github.com/sclorg/s2i-python-container/blob/master/3.5/README.md.
      iconClass: icon-python
      openshift.io/display-name: Python 3.5
      openshift.io/provider-display-name: Red Hat, Inc.
      sampleRepo: https://github.com/openshift/django-ex.git
      supports: python:3.5,python
      tags: builder,python
      version: "3.5"
    from:
      kind: DockerImage
      name: registry.access.redhat.com/rhscl/python-35-rhel7:latest
    generation: 3
    importPolicy: {}
    name: "3.5"
    referencePolicy:
      type: Source
  - annotations: null
    from:
      kind: DockerImage
      name: registry.access.redhat.com/rhscl/python-36-rhel7:latest
    generation: 6
    importPolicy: {}
    name: "3.6"
```
In the example, the annotations for the 3.6 image is listed as "null" whereas the annotations for the 3.5 image contains a description, iconClass, openshift.io parameters, tags, and version parameters.  All of which are required to add the image to the application catalog in the web console.  The Image Steam definition can be downloaded using the following command:
```
oc get is python -o yaml > python.yml
```
Please note that the Image Stream definition can also by downloaded in the json format using the "-o json" option instead.

Using vim or an editor of one's choice, modify the annotation for the Python 3.6 block in the python.yml file.  The new annotation for the 3.6 tag could be similar to the following:
```
    referencePolicy:
      type: Source
  - annotations:
      description: Build and run Python 3.6 applications on RHEL 7. For more information 
        about using this builder image, including OpenShift considerations, see https://github.com/sclorg/s2i-python
-container/blob/master/3.6/README.md.
      iconClass: icon-python
      openshift.io/display-name: Python 3.6
      openshift.io/provider-display-name: Red Hat, Inc.
      sampleRepo: https://github.com/openshift/django-ex.git
      supports: python:3.6,python
      tags: builder,python
      version: "3.6"
    from:
      kind: DockerImage
      name: registry.access.redhat.com/rhscl/python-36-rhel7:latest
    generation: 6
    importPolicy: {}
    name: "3.6"
```

## Add to Application Catalog

The update Image Stream definition can then be uploaded into OpenShift:
```
oc replace -f python.yml
```

At this point, Python 3.6 will be available via the web console.  Users will be able to select the 3.6 version of Python when browsing the application catalog.  

