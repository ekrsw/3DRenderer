# 500 行以下 3D モデラー

エリックはソフトウェア開発者であり、2D および 3D のコンピューターグラフィックス愛好家である。 ビデオゲーム、3D 特殊効果ソフトウェア、コンピューター支援設計ツールに携わってきた。 現実をシミュレートすることなら、彼はもっと学びたいと思っている。 彼のオンラインサイトは erickdransch.com。

## Introduction

人間は生来、創造的である。 斬新で、便利で、面白いものをデザインし、作り続ける。 現代では、私たちは設計や創造のプロセスを支援するソフトウェアを書いている。 コンピュータ支援設計（CAD）ソフトウェアによって、クリエイターは、物理的なバージョンを作る前に、建物、橋、ビデオゲームのアート、映画のモンスター、3D プリント可能なオブジェクト、その他多くのものを設計することができます。

CAD ツールの核心は、3 次元の設計を 2 次元の画面上で閲覧・編集できるものに抽象化する手法である。 この定義を満たすために、CAD ツールは 3 つの基本的な機能を提供しなければならない。 第一に、設計対象物を表現するためのデータ構造を備えていなければならない。これは、ユーザーが構築しようとしている 3 次元世界をコンピューターが理解するためのものである。 第二に、CAD ツールはユーザーの画面上にデザインを表示する何らかの方法を提供しなければならない。 ユーザーは 3 次元の物理的な物体を設計しているが、コンピューターの画面には 2 次元しかない。 CAD ツールは、私たちが物体をどのように認識しているかをモデル化し、ユーザーが物体の 3 次元すべてを理解できるような方法で画面に描画しなければなりません。 第三に、CAD ツールは設計されているオブジェクトと対話する方法を提供しなければならない。 ユーザーは希望する結果を得るために、設計に追加したり修正したりできなければならない。 さらに、すべてのツールは、ユーザーが共同作業を行い、共有し、保存できるように、デザインをディスクから保存し、ロードする方法が必要である。

領域特化型 CAD ツールは、その領域特有の要件に対応する多くの追加機能を提供する。 例えば、建築 CAD ツールは、建物にかかる気候ストレスをテストする物理シミュレーションを提供し、3D プリントツールは、オブジェクトが実際にプリントするのに有効かどうかをチェックする機能を備え、電気 CAD ツールは、銅の中を流れる電気の物理シミュレーションを行い、映画特殊効果スイートは、パイロキネティクスを正確にシミュレートする機能を含んでいます。

しかし、すべての CAD ツールは、少なくとも上述の 3 つの機能、すなわち設計を表現するデータ構造、それをスクリーンに表示する機能、設計と対話する方法を含んでいなければならない。

それを念頭に置いて、500 行の Python で 3D デザインを表現し、スクリーンに表示し、それを操作する方法を探ってみよう。

## Rendering as a Guide

3D モデラーにおける多くの設計決定の原動力は、レンダリング処理です。 複雑なオブジェクトをデザインに格納し、レンダリングできるようにしたいが、レンダリング・コードの複雑さは抑えたい。 レンダリング処理を検討し、シンプルなレンダリング・ロジックで任意に複雑なオブジェクトを格納・描画できるデザインのデータ構造を探ってみましょう。

### インターフェイスと Main Loop の管理

レンダリングを始める前に、いくつか設定しなければならないことがあります。 まず、デザインを表示するウィンドウを作成する必要があります。 次に、グラフィックス・ドライバと通信してスクリーンにレンダリングします。 グラフィックス・ドライバと直接通信することは避けたいので、OpenGL と呼ばれるクロスプラットフォームの抽象化レイヤと、GLUT（OpenGL Utility Toolkit）と呼ばれるライブラリを使用してウィンドウを管理します。

#### A Note About OpenGL

OpenGL は、クロスプラットフォーム開発のためのグラフィカル・アプリケーション・プログラミング・インターフェースです。 これは、プラットフォーム間でグラフィックス・アプリケーションを開発するための標準 API です。 OpenGL には大きく分けて 2 つの種類がある： レガシー OpenGL とモダン OpenGL です。

OpenGL のレンダリングは、頂点と法線で定義されたポリゴンに基づいている。 例えば、立方体の一辺をレンダリングするには、4 つの頂点とその辺の法線を指定します。

レガシー OpenGL は「固定関数パイプライン」を提供します。 グローバル変数を設定することで、プログラマーは、ライティング、カラーリング、フェイスカリングなどの機能の自動実装を有効にしたり無効にしたりできる。 OpenGL は、有効化された機能を使って自動的にシーンをレンダリングします。 この機能は非推奨です。

一方、モダン OpenGL は、プログラマが専用のグラフィックハードウェア（GPU）上で動作する「シェーダ」と呼ばれる小さなプログラムを記述するプログラマブルレンダリングパイプラインを特徴としている。 Modern OpenGL のプログラム可能なパイプラインは、Legacy OpenGL に取って代わりました。

このプロジェクトでは、非推奨であるにもかかわらず、Legacy OpenGL を使用しています。 レガシー OpenGL が提供する固定機能は、コードサイズを小さく保つのに非常に便利だ。 必要な線形代数の知識が減り、書くコードも単純になる。

#### About GLUT

OpenGL にバンドルされている GLUT を使えば、オペレーティング・システムのウィンドウを作成し、ユーザー・インターフェースのコールバックを登録することができる。 この基本的な機能で十分である。 もしウィンドウ管理やユーザーとのインタラクションのためにもっとフル機能のライブラリが必要なら、GTK や Qt のようなフルウィンドウツールキットの使用を検討するだろう。

#### The Viewer

GLUT と OpenGL の設定を管理し、モデラーの残りの部分を動かすために、Viewer というクラスを作ります。 単一の Viewer インスタンスを使用し、ウィンドウの作成とレンダリングを管理し、プログラムのメインループを含む。 Viewer の初期化処理では、GUI ウィンドウを作成し、OpenGL を初期化する。

init_interface 関数は、モデラーがレンダリングされるウィンドウを作成し、デザインがレンダリングされる必要があるときに呼び出される関数を指定します。 init_opengl 関数はプロジェクトに必要な OpenGL の状態を設定します。 マトリクスを設定し、バックフェイスカリングを有効にし、シーンを照らすライトを登録し、オブジェクトに色を付けたいことを OpenGL に伝えます。 init_scene 関数は Scene オブジェクトを作成し、初期ノードを配置します。 Scene データ構造については、後ほど詳しく説明する。 最後に、init_interaction は、後で説明するように、ユーザーとのインタラクションのためのコールバックを登録する。

Viewer を初期化した後，glutMainLoop を呼び出してプログラムの実行を GLUT に移す． この関数は決して戻らない． GLUT イベントに登録したコールバックは、それらのイベントが発生したときに呼び出されます。

```
class Viewer(object):
    def __init__(self):
        """ Initialize the viewer. """
        self.init_interface()
        self.init_opengl()
        self.init_scene()
        self.init_interaction()
        init_primitives()

    def init_interface(self):
        """ initialize the window and register the render function """
        glutInit()
        glutInitWindowSize(640, 480)
        glutCreateWindow("3D Modeller")
        glutInitDisplayMode(GLUT_SINGLE | GLUT_RGB)
        glutDisplayFunc(self.render)

    def init_opengl(self):
        """ initialize the opengl settings to render the scene """
        self.inverseModelView = numpy.identity(4)
        self.modelView = numpy.identity(4)

        glEnable(GL_CULL_FACE)
        glCullFace(GL_BACK)
        glEnable(GL_DEPTH_TEST)
        glDepthFunc(GL_LESS)

        glEnable(GL_LIGHT0)
        glLightfv(GL_LIGHT0, GL_POSITION, GLfloat_4(0, 0, 1, 0))
        glLightfv(GL_LIGHT0, GL_SPOT_DIRECTION, GLfloat_3(0, 0, -1))

        glColorMaterial(GL_FRONT_AND_BACK, GL_AMBIENT_AND_DIFFUSE)
        glEnable(GL_COLOR_MATERIAL)
        glClearColor(0.4, 0.4, 0.4, 0.0)

    def init_scene(self):
        """ initialize the scene object and initial scene """
        self.scene = Scene()
        self.create_sample_scene()

    def create_sample_scene(self):
        cube_node = Cube()
        cube_node.translate(2, 0, 2)
        cube_node.color_index = 2
        self.scene.add_node(cube_node)

        sphere_node = Sphere()
        sphere_node.translate(-2, 0, 2)
        sphere_node.color_index = 3
        self.scene.add_node(sphere_node)

        hierarchical_node = SnowFigure()
        hierarchical_node.translate(-2, 0, -2)
        self.scene.add_node(hierarchical_node)

    def init_interaction(self):
        """ init user interaction and callbacks """
        self.interaction = Interaction()
        self.interaction.register_callback('pick', self.pick)
        self.interaction.register_callback('move', self.move)
        self.interaction.register_callback('place', self.place)
        self.interaction.register_callback('rotate_color', self.rotate_color)
        self.interaction.register_callback('scale', self.scale)

    def main_loop(self):
        glutMainLoop()

if __name__ == "__main__":
    viewer = Viewer()
    viewer.main_loop()
```

レンダー関数に入る前に、線形代数について少し触れておこう。

### Coordinate Space

座標空間とは、原点と 3 つの基底ベクトル（通常は x 軸, y 軸, z 軸）の集合である。

### Point

3 次元の任意の点は、原点からの x, y, z 方向のオフセットとして表現できる。 点の表現は、その点がいる座標空間に対する相対的なものです。 同じ点でも、座標空間が違えば表現が異なる。 3 次元の任意の点は、任意の 3 次元座標空間で表現できる。

### ベクトル

ベクトルとは、x 軸, y 軸 , z 軸の各軸における 2 点間の差を表す x, y, z の値である。

### Transformation Matrix

コンピュータグラフィックスでは、点の種類に応じて複数の異なる座標空間を使い分けると便利です。 変換行列は、点をある座標空間から別の座標空間に変換します。 ベクトル 𝑣 をある座標空間から別の座標空間に変換するには、変換行列 M をかけます : v′=Mv′。 一般的な変換行列には、平行移動、スケーリング、回転があります。

### Model, World, View, and Projection Coordinate Spaces

<img src="static/newtranspipe.png" alt="Figure 13.1 - Transformation Pipeline">
アイテムをスクリーンに描くには、いくつかの異なる座標空間間で変換する必要がある。
図13.11の右側は、Eye SpaceからViewport Spaceへのすべての変換を含め、すべてOpenGLによって処理されます。
Eye Spaceから均質なクリップ空間への変換はgluPerspectiveによって処理され、正規化されたデバイス空間とビューポート空間への変換はglViewportによって処理されます。 これら2つの行列は掛け合わされ、GL_PROJECTION行列として保存されます。 このプロジェクトでは、これらの行列がどのように機能するかについての用語や詳細を知る必要はない。
