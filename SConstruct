import os
import shutil

var = Variables()
var.AddVariables(
	("BLENDER", "path to blender executable", "blender"),
	("PYTHON3", "path to python executable", "python3"),
	("PYLINT", "path to pylint executable", "pylint3"),
	("PYCODESTYLE", "path to pycodestyle executable", "pycodestyle")
)

env = Environment(variables=var)
Help(var.GenerateHelpText(env))

# Constants within the respository
EXPORT_DIR = './tests/exports'
REFERENCE_DIR = './tests/reference-exports'


def export_blends(target, source, env): 
	if os.path.exists(EXPORT_DIR):  # Clear old exports
		shutil.rmtree(EXPORT_DIR) 
	os.makedirs(EXPORT_DIR) 
	
	if os.path.exists('./tests/.import'):
		# Ensure we don't have any data cached in godot
		shutil.rmtree('./tests/.import')  
	return systemcall("{} -b --python ./tests/scenes/export_blends.py".format(
		env['BLENDER']
	))

def systemcall(command):
	if os.system(command) != 0:
		raise Exception("Build Failed")

  
def compare_exports(target, source, env): 
	import difflib
	files_1 = {f for f in os.listdir(EXPORT_DIR) if not f.endswith('.import')}
	files_2 = {f for f in os.listdir(REFERENCE_DIR) if not f.endswith('.import')}
	
	error = False
	differ = difflib.Differ(
		linejunk=lambda x: bool(x.strip()), 
		charjunk=lambda x: x in [' ', '\t', '\n']
	)
	
	for file_name in files_1.union(files_2):
		if file_name not in files_2:
			print("File {} does not exist in path {}".format(file_name, REFERENCE_DIR))
			error = True
			continue
		if file_name not in files_1:
			print("File {} does not exist in path {}".format(file_name, EXPORT_DIR))
			error = True
			continue
	
		data1 = open(os.path.join(REFERENCE_DIR, file_name)).readlines()
		data2 = open(os.path.join(EXPORT_DIR, file_name)).readlines()
		
		for line_number, line in enumerate(differ.compare(data1, data2)):
			if not line[0] == ' ':
				error = True
				print(file_name, line.strip())
				
	if error:
		raise Exception("There are differences between the current exports and the reference exports")
	else:
		print("All Files Match")

def update_examples(target, source, env):
	if os.path.exists(REFERENCE_DIR):
		shutil.rmtree(REFERENCE_DIR)
	shutil.copytree(EXPORT_DIR, REFERENCE_DIR)


def style_test(target, source, env):
	systemcall(env["PYCODESTYLE"] + ' io_scene_godot')
	systemcall(env["PYLINT"] + ' io_scene_godot')


export = env.Command('export_blends', None, export_blends) 
compare = env.Command('compare', None, compare_exports) 
env.Command('update_examples', None, update_examples)
env.Command('style_test', None, style_test)

Depends(compare, export)

Default("compare", "style_test")
