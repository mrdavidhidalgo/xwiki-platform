/*
 * See the NOTICE file distributed with this work for additional
 * information regarding copyright ownership.
 *
 * This is free software; you can redistribute it and/or modify it
 * under the terms of the GNU Lesser General Public License as
 * published by the Free Software Foundation; either version 2.1 of
 * the License, or (at your option) any later version.
 *
 * This software is distributed in the hope that it will be useful,
 * but WITHOUT ANY WARRANTY; without even the implied warranty of
 * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU
 * Lesser General Public License for more details.
 *
 * You should have received a copy of the GNU Lesser General Public
 * License along with this software; if not, write to the Free
 * Software Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA
 * 02110-1301 USA, or see the FSF site: http://www.fsf.org.
 */

// It's assumed that Jenkins has been configured to implicitly load the vars/*.groovy libraries.
// Note that the version used is the one defined in Jenkins but it can be overridden as follows:
// @Library("XWiki@<branch, tag, sha1>") _
// See https://github.com/jenkinsci/workflow-cps-global-lib-plugin for details.

stage ('Platform Builds') {
  parallel(
    'main': {
      // Build, skipping checkstyle & revapi so that the result of the build can be sent as fast as possible
      // to the dev. However note that in // we start a build with the quality profile that checks checkstyle
      // revapi and more.
      // Configures the snapshot extension repository in XWiki in the generated distributions to make it easy for
      // developers to install snapshot extensions when they do manual tests.
      build(
        goals: 'clean deploy',
        profiles: 'legacy,integration-tests,office-tests,snapshotModules',
        properties: '-Dxwiki.checkstyle.skip=true -Dxwiki.surefire.captureconsole.skip=true -Dxwiki.revapi.skip=true')

      // Note: if an error occurs in the first build above, then an exception will be raised and this job will not
      // execute which is what we want since failures can be test flickers for ex, and it could still be interesting to
      // get a distribution to test xwiki manually.

      // Build the distributions
      build(
        goals: 'clean deploy',
        profiles: 'legacy,integration-tests,office-tests,snapshotModules',
        pom: 'xwiki-platform-distribution/pom.xml')

      // Building the various functional tests, after the distribution has been built successfully.

      // Build the Flavor Test POM, required for the pageobjects module below.
      buildFunctionalTest('pom.xml')

      // Build the Flavor Test Pageobjects required by the functional test below that need an XWiki UI
      buildFunctionalTest('xwiki-platform-distribution-flavor-test-pageobjects/pom.xml')

      // Now run all tests in parallel
      parallel(
        'flavor-test-ui': {
          // Run the Flavor UI tests
          buildFunctionalTest('xwiki-platform-distribution-flavor-test-ui/pom.xml')
        },
        'flavor-test-misc': {
          // Run the Flavor Misc tests
          buildFunctionalTest('xwiki-platform-distribution-flavor-test-misc/pom.xml')
        },
        'flavor-test-storage': {
          // Run the Flavor Storage tests
          buildFunctionalTest('xwiki-platform-distribution-flavor-test-storage/pom.xml')
        },
        'flavor-test-escaping': {
          // Run the Flavor Escaping tests
          buildFunctionalTest('xwiki-platform-distribution-flavor-test-escaping/pom.xml')
        },
        'flavor-test-selenium': {
          // Run the Flavor Selenium tests
          buildFunctionalTest('xwiki-platform-distribution-flavor-test-selenium/pom.xml')
        }
      )
    },
    'testrelease': {
      // Simulate a release and verify all is fine.
      build(
        goals: 'clean install',
        profiles: 'hsqldb,jetty,legacy,integration-tests,standalone,flavor-integration-tests,distribution',
        properties: '-DskipTests -DperformRelease=true -Dgpg.skip=true -Dxwiki.checkstyle.skip=true')
    },
    'quality': {
      // Run the quality checks
      build(
        goals: 'clean install jacoco:report',
        profiles: 'quality,legacy')
    }
  )
}

def build(map)
{
  node {
    xwikiBuild {
      mavenOpts = '-Xmx2500m -Xms512m -XX:ThreadStackSize=2048'
      if (map.goals) {
        goals = map.goals
      }
      if (map.profiles) {
        profiles = map.profiles
      }
      if (map.properties) {
        properties = map.properties
      }
      if (map.pom) {
        pom = map.pom
      }
    }
  }
}

def buildFunctionalTest(pPom)
{
  def sharedGoals = 'clean deploy'
  def sharedProfiles = 'legacy,integration-tests,jetty,hsqldb,firefox'
  def sharedPOMPrefix =
    'xwiki-platform-distribution/xwiki-platform-distribution-flavor/xwiki-platform-distribution-flavor-test'
  build(goals: sharedGoals, profiles: sharedProfiles, pom: "${sharedPOMPrefix}/${pPom}")
}