---
layout: post
title: "Ruby实现快速排序，只需一行"
category: 技术
tags: [Ruby]
---


	def quick_sort(a)
		(x=a.pop) ? quick_sort(a.select{|i| i <= x}) + [x] + quick_sort(a.select{|i| i > x}) : []
	end

