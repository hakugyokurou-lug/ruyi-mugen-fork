DISTROMAP = [
    'oE2309-x86_64': ['label': 'openEuler2309 && x86_64'],
    'oE2309-riscv64': ['label': 'openEuler2309 && riscv64'],
    'ubuntu2204-x86_64': ['label': 'ubuntu2204 && x86_64'],
    'ubuntu2204-riscv64': ['label': 'ubuntu2204 && riscv64'],
    'fedora38-x86_64': ['label': 'fedora38 && x86_64'],
    'fedora38-riscv64': ['label': 'fedora38 && riscv64'],
    'revyos-riscv64': ['label': 'revyos && riscv64'],
    'debian12-x86_64': ['label': 'debian12 && x86_64'],
    'debian12-aarch64': ['label': 'debian12 && aarch64'],
    'debiansid-riscv64': ['label': 'debiansid && riscv64'],
    'archlinux-x86_64': ['label': 'archlinux && x86_64'],
    'archlinux-riscv64': ['label': 'archlinux && riscv64']
    ]

def mugen_install () {
    
    sh 'bash dep_install.sh'
    
    sh 'mkdir conf'
    sh '''
    echo '{ "NODE": [{ "ID": 1, \
    "LOCALTION": "local", \
    "MACHINE": "physical", \
    "IPV6": "::1", \
    "FRAME": "", \
    "NIC": "", \
    "MAC": "", \
    "IPV4": "127.0.0.1", \
    "USER": "", \
    "PASSWORD": "", \
    "SSH_PORT": 22, \
    "BMC_IP": "", \
    "BMC_USER": "", \
    "BMC_PASSWORD": "" }]}' >> conf/env.json
    '''
}

def mugen_run () {
    sh 'bash ruyi_mugen.sh -f ruyi -x 2>&1 | tee report_gen_tmpl/26test_log.md'
    sh 'sudo chown -R $USER:$USER ./* ./.*'
}

def mugen_report() {
    sh 'bash dep_install.sh -j'
    sh "bash report_gen.sh ${DIRSTO}"
    sh "mkdir test-artifacts"
    sh "mv ruyi-test-logs.tar.gz test-artifacts/ruyi-test-${DIRSTO}-logs.tar.gz"
    sh "mv ruyi-test-logs_failed.tar.gz test-artifacts/ruyi-test-${DIRSTO}-logs_failed.tar.gz"
    sh "mv ruyi_report/*.md test-artifacts"
}

pipeline {
    agent none

    options {
        timeout(time: 24, unit: 'HOURS')
    }

    stages {
        stage('ruyi-test') {
            matrix {
                agent { label DISTROMAP["${DIRSTO}"]['label'] }
                axes {
                    axis {
                        name "DIRSTO"
                        values 'debian12-aarch64', 'debiansid-riscv64', 'archlinux-x86_64', 'archlinux-riscv64'
                    }
                }
                
                stages {
                    stage('mugen') {
                        steps {
                            echo "Start mugen test on ${DIRSTO}"
                            mugen_install()
                            mugen_run()
                            mugen_report()
                        }
                    }
                }
                
                post {
                    always {
                        archiveArtifacts artifacts: "test-artifacts/*.*"
                        cleanWs()
                    }
                }
            }
        }
    }
}

