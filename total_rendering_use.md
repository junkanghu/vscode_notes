# how to use total-rendering

## config directory
1. yaml中的ENCODE_A代表着要不要使用appearance code
2. SPC下的：
   1. EXPAND_OFFSET:  代表间隔多少距离生成一个sparse voxel
   2. NUM_EXPANDS: 代表进行几轮expand，expand一轮会扩张许多voxel
   3. VOXEL_SIZE: 代表生成的每个voxel的边长为多少
3. DATASET下的：
   1. COLMAP_UPSCALE：代表对原始image或者说从videos中抽帧得出来的image进行downscale去做sfm，节省computatianal cost。

## 跑frame-wise的mesh（deformation结果）
step 3为生成mesh，step 4为生成texture.png
在total-rendering目录下运行：
1. export root_dir=数据集根路径（/dellnas/dataset/static_recon/Relight/junkangs/head_1124_main）
2. ```export PYTHONPATH=`pwd` ```
3. ```python dh3d_toolchains/task_generate_posed_mesh_mask.py --port_id 0 --server_port_id 0 --cfg_path $root_dir/results/xxx.yaml --ckpt_path $root_dir/results/epoch_1.ckpt  --save_dir posed --input_mesh human_final.obj```
   1. 需要修改自己的yaml文件，这里为head_1124.yaml
   2. 需要修改最终切完uv的obj文件的名字，这里为final.obj（放在数据集根路径下）
4. ```python dh3d_toolchains/task_generate_tex_4k.py --port_id 0 --server_port_id 0 --device_ids 0 --posed_dir posed --images_dir images --mesh_name final.obj --data_dir $root_dir```
   1. 需要修改mesh_name，这里为final.obj
5. 