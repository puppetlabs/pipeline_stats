#!/usr/bin/env groovy
@Library('puppet_jenkins_shared_libraries') _

import com.puppet.jenkinsSharedLibraries.BundleInstall
import com.puppet.jenkinsSharedLibraries.BundleExec

String bundleInstall(String rubyVersion) {
  def bundle_install = new BundleInstall(rubyVersion)
  return bundle_install.bundleInstall
}

String bundleExec(String rubyVersion, String command) {
  def bundle_exec = new BundleExec(rubyVersion, command)
  return bundle_exec.bundleExec
}

pipeline {
  agent { label 'worker' }
  triggers {
    // this timing needs to not overlap with any of the other jobs in this folder
    // because if one job commits traces while another is running, the commit & push
    // step won't work since we're not at the HEAD of the branch
    //
    // ref: https://jenkins.io/doc/book/pipeline/syntax/#cron-syntax
    //
    // this cron statement specifies a run between 12:00 & 2:59am Tuesdays & Saturdays
    cron('H H(0-2) * * 2,6')
  }

  environment {
    GEM_SOURCE='https://artifactory.delivery.puppetlabs.net/artifactory/api/gems/rubygems/'
    RUBY_VERSION='2.5.1'
    PIPELINE_BRANCH='dt_job_01'
    GIT_CHANGED_FILES='0' // will be overriden
    BRANCH='master'
  }

  stages {
    stage('bundle install') {
      steps {
        sh bundleInstall(env.RUBY_VERSION)
      }
    }
    stage('collect traces') {
      environment {
        PIPELINE_STATS_LOGIN_FILE=credentials('jenkins_api_client-login')
      }
      steps {
        sh bundleExec(env.RUBY_VERSION, 'collector')
      }
    }
    stage('commit new traces to project') {
      // when { environment name: 'GIT_CHANGED_FILES', value: '1' }
      steps {
        sh 'git status'
        sh 'git add build_traces'
        sh "git commit -m 'add new puppet-agent-${env.BRANCH} traces'"
        sh "git push origin ${env.PIPELINE_BRANCH}"
      }
    }
  }
}
