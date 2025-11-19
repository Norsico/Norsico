name: Update Readme

on:
  # 每天半夜自动运行
  schedule:
    - cron: "0 0 * * *" 
  # 允许你手动触发（去Actions页面点击Run workflow）
  workflow_dispatch:
  push:
    branches:
    - main

jobs:
  generate:
    runs-on: ubuntu-latest
    timeout-minutes: 10
    
    steps:
      # 1. 生成贪吃蛇 (Snake)
      - name: generate-snake-game-from-github-contribution-grid
        uses: Platane/snk/svg-only@v3
        with:
          github_user_name: ${{ github.repository_owner }}
          outputs: |
            dist/github-contribution-grid-snake.svg
            dist/github-contribution-grid-snake-dark.svg?palette=github-dark

      # 2. 生成左侧：全年的 3D 甚至日历 (Isometric Calendar)
      - name: Full year calendar (Isometric)
        uses: lowlighter/metrics@latest
        with:
          filename: dist/metrics.plugin.isocalendar.svg
          token: ${{ secrets.METRICS_TOKEN }}
          base: ""
          plugin_isocalendar: yes
          plugin_isocalendar_duration: full-year

      # 3. 生成右侧：普通日历 (Flat Calendar)
      - name: Commit calendar Current year
        uses: lowlighter/metrics@latest
        with:
          filename: dist/metrics.plugin.calendar.svg
          token: ${{ secrets.METRICS_TOKEN }}
          base: ""
          plugin_calendar: yes
          plugin_calendar_limit: 1

      # 4. 将生成的图片推送到仓库
      - name: push github-contribution-grid-snake.svg to the output branch
        uses: crazy-max/ghaction-github-pages@v3.1.0
        with:
          target_branch: output
          build_dir: dist
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
