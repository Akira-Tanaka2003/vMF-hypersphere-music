# Chord Center Visualizations / コード中心可視化まとめ

This document summarizes the chord-center visualization results for the Conformer-vMF / vMF hypersphere music representation project.  
本資料では、Conformer-vMF / vMF 超球面音楽表現におけるコード中心可視化結果を整理します。

## 1. Purpose / 目的

The purpose of this analysis is to examine how chord categories such as `template`, `triad`, and `seventh` are arranged in the learned vMF representation space.  
本分析の目的は、`template`、`triad`、`seventh` などのコードカテゴリが、学習された vMF 表現空間上でどのように配置されているかを確認することです。

The visualizations include sphere projections, circle projections, nearest-edge plots, angular-distance heatmaps, count distributions, and PCA loading plots.  
可視化には、球面射影、円上射影、近傍エッジ図、角距離ヒートマップ、出現数分布、PCA loading 図を含めています。

## 2. Included files / 掲載ファイル

### Sphere and circle visualizations / 球面・円上可視化

- `figures/chord_center_visualizations/pop1k7_1000_template_sphere_music_labels.png`
- `figures/chord_center_visualizations/pop1k7_1000_triad_sphere_music_labels.png`
- `figures/chord_center_visualizations/pop1k7_1000_seventh_sphere_music_labels.png`
- `figures/chord_center_visualizations/pop1k7_1000_template_circle_music_labels.png`
- `figures/chord_center_visualizations/pop1k7_1000_triad_circle_music_labels.png`
- `figures/chord_center_visualizations/pop1k7_1000_seventh_circle_music_labels.png`

### Angular distance and count plots / 角距離・出現数可視化

- `figures/chord_center_visualizations/pop1k7_1000_template_angular_distance_heatmap_music_labels.png`
- `figures/chord_center_visualizations/pop1k7_1000_triad_angular_distance_heatmap_music_labels.png`
- `figures/chord_center_visualizations/pop1k7_1000_seventh_angular_distance_heatmap_music_labels.png`
- `figures/chord_center_visualizations/pop1k7_1000_template_counts_bar_music_labels.png`
- `figures/chord_center_visualizations/pop1k7_1000_triad_counts_bar_music_labels.png`
- `figures/chord_center_visualizations/pop1k7_1000_seventh_counts_bar_music_labels.png`

### PCA variable explanation / PCA変数説明

- `figures/pca_variable_analysis/template_PC1_loadings.png`
- `figures/pca_variable_analysis/triad_PC1_loadings.png`
- `figures/pca_variable_analysis/seventh_PC1_loadings.png`
- `figures/pca_variable_analysis/template_sphere_with_pca_variable_labels.png`
- `figures/pca_variable_analysis/triad_sphere_with_pca_variable_labels.png`
- `figures/pca_variable_analysis/seventh_sphere_with_pca_variable_labels.png`

## 3. Main interpretation / 主な解釈

The PCA was applied to 10-dimensional vMF center vectors averaged for each chord category.  
PCA は、コードカテゴリごとに平均化された 10 次元 vMF 中心ベクトルに対して適用しました。

The 10 variables correspond to pitch-circle components, circle-of-fifths components, pitch-transition components, normalized pitch change, register, and bar-position components.  
10 変数は、音高円成分、5度圏成分、音高遷移成分、正規化音高変化量、音域、小節内位置成分に対応します。

For `template` and `triad`, PC1 is strongly related to the circle-of-fifths component, especially `fifth_x`.  
`template` と `triad` では、PC1 が特に `fifth_x` を中心とした 5度圏成分と強く関係しています。

This suggests that the differences among chord templates and triad categories may be represented partly as circle-of-fifths-related directional relationships.  
これは、コードテンプレートや三和音カテゴリの違いが、部分的に 5度圏的な方向関係として表現されている可能性を示します。

For `seventh`, PC1 is more strongly related to pitch-transition components, especially `transition_y`.  
一方、`seventh` では、PC1 が特に `transition_y` を中心とした音高遷移成分と強く関係しています。

This suggests that seventh-related categories may be represented not only as static sonorities, but also through progression, resolution, and temporal context.  
これは、7th 系カテゴリが単なる同時発音の響きとしてだけではなく、進行、解決、時間的文脈と結びついて表現されている可能性を示します。

## 4. Important caveats / 注意点

PCA components are linear combinations of the original vMF variables.  
PCA の主成分は、元の vMF 変数そのものではなく、それらの線形結合です。

The circle plots are normalized 2D PCA visualizations and should not be interpreted as the actual circle of fifths.  
円上プロットは、PCA による 2 次元化結果を単位円上に正規化した可視化であり、5度圏そのものではありません。

The chord labels are estimated from MIDI pitch sets, not manually annotated chord labels.  
コードラベルは手作業アノテーションではなく、MIDI の同時発音ピッチ集合に基づく簡易推定です。

The `none` label does not strictly mean a no-chord annotation.  
`none` は厳密な no-chord アノテーションではなく、前処理上、特定のコードテンプレートに分類されなかったイベントを表します。

## 5. Recommended usage / 推奨される使い方

These figures should be used as interpretability visualizations for the learned vMF chord-center space.  
これらの図は、学習された vMF コード中心空間の解釈可能性を確認するための可視化として用います。

The most defensible claim is not that the model discovered music theory from scratch, but that the designed vMF representation preserves and organizes chord-related information in a geometrically interpretable way.  
最も安全な主張は、モデルが音楽理論を完全にゼロから発見したということではなく、設計した vMF 表現がコード関連情報を幾何的に解釈しやすい形で保持・整理している、ということです。
