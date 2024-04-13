
 Map<Integer, Integer> indexMap = new HashMap<>();
        for (int i = 0; i < A.size(); i++) {
            indexMap.put(A.get(i), i);
        }

        // Sort list B based on the indices of elements in list A
        Collections.sort(B, Comparator.comparingInt(b -> indexMap.getOrDefault(b, Integer.MAX_VALUE)));
