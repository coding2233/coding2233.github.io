---
title: Unity Hex Map技术测试
author: 船长
date: 2020-04-27 20:44:36 +0800
categories: [Trash, Useless]
tags: [unity]
render_with_liquid: true
---
![test](/assets/images/hex.png)

HexMap常用于战棋的地形，最近有这方面的需求，做一个简单的测试，从画一个正六边形开始。

### 0x00. 正六边形
![](/assets/images/hex.png)  
* 如上图所示，两个圆，可以很规范的画一个六变形。  
* 外圆半径定为:`public const float OuterRadius = 10f;`  
* 内圆半径定位:`public const float InnerRadius = OuterRadius * 0.866025404f;`  
* 定义模型:
```csharp
Vector3[] vers = new Vector3[]{
            new Vector3(0f,0f,0f),
            new Vector3(0f, 0f, OuterRadius),
            new Vector3(InnerRadius, 0f, 0.5f * OuterRadius),
            new Vector3(InnerRadius, 0f, -0.5f * OuterRadius),
            new Vector3(0f, 0f, -OuterRadius),
            new Vector3(-InnerRadius, 0f, -0.5f * OuterRadius),
            new Vector3(-InnerRadius, 0f, 0.5f * OuterRadius)
        };
int[] triangles = new int[] { 0, 1, 2, 0, 2, 3, 0, 3, 4, 0, 4, 5, 0, 5, 6, 0, 6, 1 };
```

### 0x01. unity模型
* Hex模型
![](/assets/images/hex_mesh.png)
* Hex Grid
```csharp
 for (int i = 0; i < _column; i++)
        {
            for (int j = 0; j < _row; j++)
            {
                HexMesh clone = new GameObject().AddComponent<HexMesh>();
                float offet = j % 2 == 0 ? 0 : HexMesh.InnerRadius;
                clone.transform.localPosition = new Vector3(i * HexMesh.InnerRadius * 2 + offet, 0, j * HexMesh.OuterRadius * 1.5f);
            }
        }
```
![](/assets/images/hex_mesh_grid.png)


其实上面用来满足以纯六变形做地形的需求，是没问题的， 但是我希望我们的地形是一个现成风格更加可控可变的，HexMap应该是附加到地形之上，所以需要重新转换一下思路。

### 0x02. 八叉树管理场景的三角面
创建三角形的数据结构，保存三角行的顶点数据，以及包含三角形的Bounds
```csharp
public struct TriangleVertices
{
    public Vector3 Vertex0;
    public Vector3 Vertex1;
    public Vector3 Vertex2;

    public Bounds Bounds;

    public TriangleVertices(Vector3 vertex0, Vector3 vertex1, Vector3 vertex2)
    {
        Vertex0 = vertex0;
        Vertex1 = vertex1;
        Vertex2 = vertex2;

        float maxX = Mathf.Max(vertex0.x, vertex1.x, vertex2.x);
        float maxY = Mathf.Max(vertex0.y, vertex1.y, vertex2.y);
        float maxZ = Mathf.Max(vertex0.z, vertex1.z, vertex2.z);

        float minX = Mathf.Min(vertex0.x, vertex1.x, vertex2.x);
        float minY = Mathf.Min(vertex0.y, vertex1.y, vertex2.y);
        float minZ = Mathf.Min(vertex0.z, vertex1.z, vertex2.z);

        Vector3 si = new Vector3(maxX - minX, maxY - minY, maxZ - minZ);
        if (si.x <= 0)
            si.x = 0.1f;
        if (si.y <= 0)
            si.y = 0.1f;
        if (si.z <= 0)
            si.z = 0.1f;
        Vector3 ct = new Vector3(minX, minY, minZ) + si / 2;
        Bounds = new Bounds(ct, si);
    }

}
```
创建ScriptableObject保存数据,并在细化八叉树到最小的节点，如果有三角形被多个Bounds包含，则同时存同一个三角形
```csharp
 public struct OctreeNode
{
        public Bounds Bounds;
        public int CurrentDepth;
        public List<TriangleVertices> Triangles;
}

[System.Serializable]
public class Octree : ScriptableObject
{
    public OctreeNode Root;
    public int MaxDepth;
    public int TriangleCount;
    public void Setup(Bounds maxBounds, int maxDepth, List<TriangleVertices> triangles)
    {
        MaxDepth = maxDepth;
        Root = new OctreeNode();
        Root.Bounds = maxBounds;

        SplitBounds(ref Root, triangles, 0);
    }

    private void SplitBounds(ref OctreeNode node, List<TriangleVertices> triangles, int deep)
    {
        node.CurrentDepth = deep;
        if (deep < MaxDepth)
        {
            node.Triangles = new List<TriangleVertices>();
            node.ChildNodes = new OctreeNode[8];

            Bounds bounds = node.Bounds;
            Vector3 half2Vector = bounds.size / 4;
            // 八块空间的位置
            node.ChildNodes[0].Bounds.center = bounds.center - half2Vector;
            node.ChildNodes[0].Bounds.size = bounds.size / 2;
            node.ChildNodes[1].Bounds.center = node.ChildNodes[0].Bounds.center + new Vector3(bounds.size.x / 2, 0, 0);
            node.ChildNodes[1].Bounds.size = bounds.size / 2;
            node.ChildNodes[2].Bounds.center = node.ChildNodes[1].Bounds.center + new Vector3(0, 0, bounds.size.z / 2);
            node.ChildNodes[2].Bounds.size = bounds.size / 2;
            node.ChildNodes[3].Bounds.center = node.ChildNodes[0].Bounds.center + new Vector3(0, 0, bounds.size.z / 2);
            node.ChildNodes[3].Bounds.size = bounds.size / 2;
            node.ChildNodes[4].Bounds.center = node.ChildNodes[0].Bounds.center + new Vector3(0, bounds.size.y / 2, 0);
            node.ChildNodes[4].Bounds.size = bounds.size / 2;
            node.ChildNodes[5].Bounds.center = node.ChildNodes[1].Bounds.center + new Vector3(0, bounds.size.y / 2, 0);
            node.ChildNodes[5].Bounds.size = bounds.size / 2;
            node.ChildNodes[6].Bounds.center = node.ChildNodes[2].Bounds.center + new Vector3(0, bounds.size.y / 2, 0);
            node.ChildNodes[6].Bounds.size = bounds.size / 2;
            node.ChildNodes[7].Bounds.center = node.ChildNodes[3].Bounds.center + new Vector3(0, bounds.size.y / 2, 0);
            node.ChildNodes[7].Bounds.size = bounds.size / 2;

            //整理在自己的碰撞盒子里面
            for (int i = 0; i < triangles.Count; i++)
            {
                TriangleVertices triangle = triangles[i];
                if (!node.Bounds.Intersects(triangle.Bounds))
                {
                    triangles.RemoveAt(i);
                    i--;
                }
            }
            // 3、遍历8个区域 递归调用
            List<TriangleVertices> childtriangles = new List<TriangleVertices>();
            for (int i = 0; i < node.ChildNodes.Length; i++)
            {
                childtriangles.Clear();
                node.ChildNodes[i].Triangles = new List<TriangleVertices>();
                for (int j = 0; j < triangles.Count; j++)
                {
                    TriangleVertices triangle = triangles[j];
                    if (node.ChildNodes[i].Bounds.Intersects(triangle.Bounds))
                    {
                        childtriangles.Add(triangle);
                    }
                }

                if (childtriangles.Count > 0)
                {
                    SplitBounds(ref node.ChildNodes[i], childtriangles, node.CurrentDepth + 1);
                }
            }


        }
        else
        {
            node.Triangles.AddRange(triangles);
            TriangleCount += triangles.Count;
        }
    }
}
```

### 0x03. 创建独立的检索Bounds
利用独立的Bounds，去检索Octree里的bounds，并获取对应的三角形的数据，动态生成模型，然后跟其顶点在Bounds的范围，来修改模型的UV数据，保证其值在0-1之间。
```csharp
public void Handle(Bounds bounds, IProjectorHandle handle, string meshName = null)
{
    List<TriangleVertices> triangles = ListPool<TriangleVertices>.Get();
    //索取所有的三角形
    GetTriangles(bounds, this, triangles);
    //转换成模式
    Mesh mesh = new Mesh();
    if (!string.IsNullOrEmpty(meshName))
        mesh.name = $"{meshName}";
    List<Vector3> vertices = ListPool<Vector3>.Get();
    List<Vector2> uvs = ListPool<Vector2>.Get();
    List<int> indexs = ListPool<int>.Get();
    Vector2 uv = new Vector2();
    int index = 0;
    for (int i = 0; i < triangles.Count; i++)
    {
        TriangleVertices triangle = triangles[i];
        //vertices
        vertices.Add(triangle.Vertex0);
        vertices.Add(triangle.Vertex1);
        vertices.Add(triangle.Vertex2);
        //uvs
        uv.x = (triangle.Vertex0.x - bounds.min.x) / bounds.size.x;
        uv.y = (triangle.Vertex0.z - bounds.min.z) / bounds.size.z;
        uvs.Add(uv);
        uv.x = (triangle.Vertex1.x - bounds.min.x) / bounds.size.x;
        uv.y = (triangle.Vertex1.z - bounds.min.z) / bounds.size.z;
        uvs.Add(uv);
        uv.x = (triangle.Vertex2.x - bounds.min.x) / bounds.size.x;
        uv.y = (triangle.Vertex2.z - bounds.min.z) / bounds.size.z;
        uvs.Add(uv);
        //index
        indexs.Add(index);
        indexs.Add(index + 1);
        indexs.Add(index + 2);
        index += 3;
    }
    //设置模型信息
    mesh.SetVertices(vertices);
    mesh.SetUVs(0, uvs);
    mesh.SetTriangles(indexs, 0);
    ListPool<TriangleVertices>.Release(triangles);
    ListPool<Vector3>.Release(vertices);
    ListPool<Vector2>.Release(uvs);
    ListPool<int>.Release(indexs);
    //回调
    handle.ProjectorHandle(mesh);
}
```

### 0x04. 准备一个包含六变形的图片的材质
取消掉`Generate Mip Maps`，`Wrap Mode`改为`Clamp`
![](/assets/images/hex_material.png)


### 0x05. 生成多个Hex
不断移动独立Bounds，即可生成多个Hex组成HexMap
```csharp
/// <summary>
/// 创建模型的GameObject
/// </summary>
private void MakeMeshGameObject()
{
    //projetor的大小
    float size = _octreeConfig.ProjectorSize;
    float halfSize = size * 0.5f;
    float offsetSize = _octreeConfig.ProjectorSize * _octreeConfig.ProjectorInnerCircleScale;
    float offsetHalfSize = offsetSize * 0.5f;

    //初始化 projector的数据    
    Vector3 startPos = new Vector3(_octreeConfig.MaxBounds.min.x + halfSize, _octreeConfig.MaxBounds.center.y, _octreeConfig.MaxBounds.min.z + halfSize);
    Vector3 projectorSize = new Vector3(size, _octreeConfig.MaxBounds.size.y, size);
    //初始化 projectorBounds
    for (int i = 0; i < _octreeConfig.Rows; i++)
    {
        for (int j = 0; j < _octreeConfig.Columns; j++)
        {
            float offset = i % 2 == 0 ? 0 : offsetHalfSize;
            Vector3 center = startPos + new Vector3(j * offsetSize + offset, 0, i * halfSize * 1.5f);
            Bounds projectorBounds = new Bounds(center, projectorSize);
            _octreeDebug.ProjectorBounds = projectorBounds;
            _octree.Root.Handle(projectorBounds, this, $"{i}_{j}");
            // new Task(
            //     () =>
            //     {
            //         _octree.Root.Handle(projectorBounds, this);
            //     }
            // ).Start();
        }
    }
}
```
![](/assets/images/hex_scene_mesh.png)


### 0x05. 导出FBX
使用unity官方插件`FBX Exporter`导出模型文件，可作为静态模型


### 0x06. TODO
- [ ] 生成模型多线程优化
- [ ] HexMap模型优化
- [ ] HexMap管理，碰撞检测
- [ ] DrawCall优化
