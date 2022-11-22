# plugin.pb.h

```cpp
#include <google/protobuf/compiler/plugin.pb.h>
namespace google::protobuf::compiler
```

protocプラグイン用のAPI。

このファイルでは、プロトコードジェネレータプラグインのAPIを構成するプロトコルメッセージクラスのセットを定義します。C++ で書かれたプラグインは、おそらく protobuf レベルの API を扱う代わりに plugin.h の API を基に構築すべきですが、他の言語のプラグインは以下で定義されているように生のメッセージを扱う必要があるでしょう。

プロトコルコンパイラは現在、自動生成されたドキュメントをサポートしていないので、このページには何も記述がありません。このファイルは、プロトコルコンパイラが plugin.proto から生成したもので、その内容は次のとおりです。




```proto
/*
  Protocol Buffers - Google's data interchange format
  Copyright 2008 Google Inc.  All rights reserved.
  https://developers.google.com/protocol-buffers/
  
  ソースコード形式およびバイナリ形式での再配布および使用は、変更の有無にかかわらず、以下の条件を満たす場合に限り許可されます。
  
  * ソースコードを再配布する場合は、上記の著作権表示、この条件一覧、および以下の免責事項を保持する必要があります。
  
  * バイナリ形式で再配布する場合は、上記の著作権表示、この条件一覧、および以下の免責事項を、配布物とともに提供される文書および/またはその他の資料で再現する必要があります。
  
  * Google Inc.の名前およびその貢献者の名前は、書面による事前の特別な許可なしに、このソフトウェアから派生した製品を推奨または宣伝するために使用することはできません。
  
  本ソフトウェアは、著作権所有者および貢献者によって「現状のまま」提供され、商品性および特定目的への適合性の黙示保証を含むがこれに限定されない、いかなる明示または黙示の保証も放棄される。本ソフトウェアの使用により発生した直接的、間接的、偶発的、特別、典型的、または結果的な損害（代替品またはサービスの調達、使用、データ、または利益の損失、または事業の中断を含むがこれに限定されない）に対して、その原因が契約、厳格責任または不法行為（過失またはその他）のいずれであっても、たとえその損害発生の可能性について知らされていたとしても、いかなる責任論によっても著作権者および貢献者が責任を負うものではありません。
*/

/*
  作者: kenton@google.com (Kenton Varda)
  
  警告：このプラグインのインターフェースは実験的なものであり、将来的に変更される可能性があります。
  
  protocはプラグインにより拡張することができます。プラグインとは、標準入力からCodeGeneratorRequestを読み、標準出力にCodeGeneratorResponseを書き出すだけのプログラムです。
  
  C++でプラグインを書く場合は、このファイルの定義を使う代わりに、google/protobuf/compiler/plugin.hをインクルードして使用することもできます。
  
  "protoc-gen-$NAME" というファイル名でプラグインの実行ファイルを作成したら、パスの通ったところにそれを配置してください。そのあと、protocで "--${NAME}_out" のように指定すればプラグインが使えます。
*/

syntax = "proto2";

package google.protobuf.compiler;
option java_package = "com.google.protobuf.compiler";
option java_outer_classname = "PluginProtos";

option go_package = "google.golang.org/protobuf/types/pluginpb";

import "google/protobuf/descriptor.proto";

// プロトコルコンパイラのバージョン番号
message Version {
  optional int32 major = 1;
  optional int32 minor = 2;
  optional int32 patch = 3;
  // alpha、beta、rc releaseなどの文字列を格納。安定版リリースの場合は空文字列。
  optional string suffix = 4;
}

// プラグインの標準入力には、エンコードされた CodeGeneratorRequest が渡される
message CodeGeneratorRequest {
  /*
    ユーザがコマンドラインで指定した .proto ファイルのファイルパス
    各ファイルのディスクリプタは、以下のproto_fileに含まれる。
  */
  repeated string file_to_generate = 1;

  /*
    コマンドラインから渡されたジェネレータパラメータ
  */
  optional string parameter = 2;

  /*
    files_to_generate に含まれるすべてのファイルと、それらがインポートするすべてのファイルに対する FileDescriptorProtos。 ファイルはトポロジー順で表示され、各ファイルはそれをインポートするファイルの前に表示されます。

    protoc は全ての proto_files が上記のフィールドの後に書き込まれることを保証しています、たとえこれが技術的に protobuf のワイヤーフォーマットによって保証されていないとしても。 これは理論的には、プラグインが FileDescriptorProtos をストリームして、一度にメモリに全セットを読み込むのではなく、一つずつ処理することを可能にするものである。 しかし、この記事を書いている時点では、protoc 側で同様に最適化されていません -- それは、プラグインに送信する前にすべてのフィールドを一度にメモリに格納します。
    
    FileDescriptorProto のフィールドと拡張子の型名は、常に完全修飾されます。
  */
  repeated FileDescriptorProto proto_file = 15;

  // プロトコルコンパイラのバージョン番号
  optional Version compiler_version = 3;

}

// プラグインは、エンコードしたCodeGeneratorResponseを標準出力に書き出すこと
message CodeGeneratorResponse {
  /*
    エラーメッセージ
    
    .protoファイルの解析に失敗したときはここに何らかの文字列を格納し、ステータスコード0で終了すること。
    
    protoc 自体の問題を示すエラー（たとえば入力されたCodeGeneratorRequestが解析不能であるなど）のときは、標準エラーにメッセージを出力し、非0のステータスコードで終了すること
  */
  optional string error = 1;

  // A bitmask of supported features that the code generator supports.
  // This is a bitwise "or" of values from the Feature enum.
  optional uint64 supported_features = 2;

  // Sync with code_generator.h.
  enum Feature {
    FEATURE_NONE = 0;
    FEATURE_PROTO3_OPTIONAL = 1;
  }

  // Represents a single generated file.
  message File {
    // The file name, relative to the output directory.  The name must not
    // contain "." or ".." components and must be relative, not be absolute (so,
    // the file cannot lie outside the output directory).  "/" must be used as
    // the path separator, not "\".
    //
    // If the name is omitted, the content will be appended to the previous
    // file.  This allows the generator to break large files into small chunks,
    // and allows the generated text to be streamed back to protoc so that large
    // files need not reside completely in memory at one time.  Note that as of
    // this writing protoc does not optimize for this -- it will read the entire
    // CodeGeneratorResponse before writing files to disk.
    optional string name = 1;

    // If non-empty, indicates that the named file should already exist, and the
    // content here is to be inserted into that file at a defined insertion
    // point.  This feature allows a code generator to extend the output
    // produced by another code generator.  The original generator may provide
    // insertion points by placing special annotations in the file that look
    // like:
    //   @@protoc_insertion_point(NAME)
    // The annotation can have arbitrary text before and after it on the line,
    // which allows it to be placed in a comment.  NAME should be replaced with
    // an identifier naming the point -- this is what other generators will use
    // as the insertion_point.  Code inserted at this point will be placed
    // immediately above the line containing the insertion point (thus multiple
    // insertions to the same point will come out in the order they were added).
    // The double-@ is intended to make it unlikely that the generated code
    // could contain things that look like insertion points by accident.
    //
    // For example, the C++ code generator places the following line in the
    // .pb.h files that it generates:
    //   // @@protoc_insertion_point(namespace_scope)
    // This line appears within the scope of the file's package namespace, but
    // outside of any particular class.  Another plugin can then specify the
    // insertion_point "namespace_scope" to generate additional classes or
    // other declarations that should be placed in this scope.
    //
    // Note that if the line containing the insertion point begins with
    // whitespace, the same whitespace will be added to every line of the
    // inserted text.  This is useful for languages like Python, where
    // indentation matters.  In these languages, the insertion point comment
    // should be indented the same amount as any inserted code will need to be
    // in order to work correctly in that context.
    //
    // The code generator that generates the initial file and the one which
    // inserts into it must both run as part of a single invocation of protoc.
    // Code generators are executed in the order in which they appear on the
    // command line.
    //
    // If |insertion_point| is present, |name| must also be present.
    optional string insertion_point = 2;

    // The file contents.
    optional string content = 15;

    // Information describing the file content being inserted. If an insertion
    // point is used, this information will be appropriately offset and inserted
    // into the code generation metadata for the generated files.
    optional GeneratedCodeInfo generated_code_info = 16;
  }
  repeated File file = 15;
}
```

このファイルのクラス
ファイルメンバー
これらの定義は、クラスの一部ではありません。
const ::protobuf::internal::DescriptorTable	
descriptor_table_google_2fprotobuf_2fcompiler_2fplugin_2eproto
PROTOBUF_NAMESPACE_CLOSE PROTOBUF_NAMESPACE_OPEN protobuf::compiler::CodeGeneratorRequest *	
Arena::CreateMaybeMessage< protobuf::compiler::CodeGeneratorRequest >(Arena * )
protobuf::compiler::CodeGeneratorResponse *	
Arena::CreateMaybeMessage< protobuf::compiler::CodeGeneratorResponse >(Arena * )
protobuf::compiler::CodeGeneratorResponse_File *	
Arena::CreateMaybeMessage< protobuf::compiler::CodeGeneratorResponse_File >(Arena * )
protobuf::compiler::Version *	
Arena::CreateMaybeMessage< protobuf::compiler::Version >(Arena * )
const EnumDescriptor *	
GetEnumDescriptor< protobuf::compiler::CodeGeneratorResponse_Feature >()
