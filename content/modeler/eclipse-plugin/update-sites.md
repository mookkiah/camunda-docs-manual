---

title: 'Eclipse Plugin Update Sites'
weight: 10


menu:
  main:
    name: "Update Sites"
    identifier: "eclipse-plugin-update-sites"
    parent: "eclipse-plugin"
    pre: "Select the correct update site for installing the eclipse plugin."

---

This page explains which update site to choose for installing the Eclipse plugin.
The update site depends on

* the version of Camunda Process Engine you are using
* the version of Eclipse you are using

**Right click on the links in the Table below** to copy the correct update site URL.

<table class="table" style="font-size: 17px">
  <tr>
    <td>Process Engine Version</td>
    <td>Eclipse Kepler</td>
    <td>Eclipse Indigo / Juno</td>
  </tr>
  <tr>
    <td><strong>7.4.0+</strong>, 7.3.3+, 7.2.6+</td>
    <td>
      <a href="http://camunda.org/release/camunda-eclipse-plugin/update-sites/kepler/archive/3.0.0/">3.0.0</a> or
      <a href="http://camunda.org/release/camunda-eclipse-plugin/update-sites/kepler/latest/site/">latest</a>
    <td>
      <a href="http://camunda.org/release/camunda-eclipse-plugin/update-sites/indigo/archive/3.0.0/">3.0.0</a> or
      <a href="http://camunda.org/release/camunda-eclipse-plugin/update-sites/indigo/latest/site/">latest</a>
  </tr>
  <tr>
    <td>Older versions
    <td>
      <a href="http://camunda.org/release/camunda-eclipse-plugin/update-sites/kepler/archive/2.7.1/">2.7.1</a>
    </td>
    <td>
      <a href="http://camunda.org/release/camunda-eclipse-plugin/update-sites/indigo/archive/2.7.1/">2.7.1</a>
    </td>
  </tr>
</table>

{{< note class="warning" title="Latest Version" >}}
Using the "Latest Version" update sites may cause you to install an Eclipse Plugin version which is incompatible with
your Process Engine version. See Section "Automatic Updates" below for details.
{{< /note >}}

# Automatic Updates with "Latest"

The "Latest Version" update sites always point to the latest released version of the Eclipse plugin. Choosing these update sites has the advantage that if you run the "Check for Updates" Wizard in Eclipse, you will be able to update to a new version of the Camunda Eclipse plugin. However, this advantage can also lead to problems: if you update to a newer version of the Eclipse modeler without updating the process engine as well, the Eclipse plugin may produce models which are incompatible with the version of the process engine you use.
In case you accidentally updated to an incompatible version of the Eclipse plugin, you have to downgrade again. See the corresponding section on "Downgrading the Eclipse Plugin" further down on this page.

# Downgrading the Eclipse Plugin

If you installed an incompatible version of the Eclipse plugin (version of Eclipse plugin not matching Camunda process engine version) then you have to downgrade the Eclipse plugin.

To downgrade the Eclipse plugin, the following steps are necessary:

1. Uninstall the incompatible version of the Eclipse plugin,
2. Change the update site URL to point to the correct version (See table at the top of this page),
3. Install a compatible version from the update site.