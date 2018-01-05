import os
import json
from json import JSONEncoder

class MyEncoder(JSONEncoder):
    def default(self, o):
        return str(o)

env = Environment()
env.AppendENVPath('PATH', '/sbin')
builder_repo = ARGUMENTS.get('builder_repo', '.')
buildlist = ARGUMENTS.get('buildlist', './buildlist')
with open(buildlist, 'r') as buildlist_file:
    targets = [x.strip() for x in buildlist_file.readlines()]

root_path = builder_repo + '/containers/'

container_nodes = []

struct = {}
for t in targets:
    path, name = os.path.split(t)
    for i in range(len(path.split('/'))):
        prefix = '/'.join(path.split('/')[:i])
        if prefix not in struct:
            struct[prefix] = {}
    if path not in struct:
        struct[path] = {}
    di = env.Command('virtual_' + t, root_path + t + '/Dockerfile', 'cd ' + builder_repo + '/containers && ./build.sh ' + root_path + t)
    #di = env.Command('virtual_' + t, root_path + t + '/Dockerfile', 'echo invoked dummy builder $SOURCE $TARGET >/dev/null')
    container_nodes.append(di)
    struct[path][name] = di

for path, containers in struct.items():
    if 'base' not in containers:
        print 'Adding dummy base container in |' + path + '|'
        containers['base'] = env.Command('virtualbase_' + path, [], action='echo xxx')

for path, containers in struct.items():
    parent = os.path.split(path)[0]
    print 'Processing dir |' + path +'| with parent |' + parent + '|' 
    if parent != path and 'base' in struct.get(parent, {}):
        env.Depends(containers['base'], struct[parent]['base'])
    for c in containers:
        if c != 'base':
            env.Depends(containers[c], containers['base'])

print json.dumps(struct, indent=4, cls=MyEncoder)

#root_node = env.Alias('build', container_nodes)
env.Default(container_nodes)
